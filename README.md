# LAB-5-KELAS-SIR
//Akmal Farhan Bin Ahmad Annuar (2022660498)
//Nurul Iffah Hasyimah Binti Hasan (2022457982)
//Zuhairah Nurhidayah Binti Zainudin (2022457722)  
//Nurfadliyana 'Aqilah Binti Sulaiman (2022824754)  

//Code Main.Dart
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        primaryColor: Color(0xFF1E3F63),
        hintColor: Colors.black,
        scaffoldBackgroundColor: Colors.white,
        fontFamily: 'Roboto',
        textTheme: TextTheme(
          headline6: TextStyle(
              fontSize: 24.0,
              fontWeight: FontWeight.bold,
              color: Color(0xFF1E3F63)),
          bodyText2: TextStyle(fontSize: 16.0, color: Colors.black87),
        ),
        elevatedButtonTheme: ElevatedButtonThemeData(
          style: ElevatedButton.styleFrom(
            primary: Color(0xFF1E3F63),
            onPrimary: Colors.white,
            textStyle: TextStyle(fontSize: 18.0),
            padding: EdgeInsets.symmetric(vertical: 12.0),
          ),
        ),
        textButtonTheme: TextButtonThemeData(
          style: TextButton.styleFrom(
            primary: Color(0xFF1E3F63),
            textStyle: TextStyle(fontSize: 16.0),
          ),
        ),
        appBarTheme: AppBarTheme(
          color: Color(0xFF1E3F63),
        ),
      ),
      home: LoginPage(),
    );
  }
}

class MyAppInfo extends StatelessWidget {
  final User user;

  MyAppInfo(this.user);

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 2,
      child: Scaffold(
        appBar: AppBar(
          title: Text("User Information"),
          bottom: TabBar(
            tabs: [
              Tab(text: "User Info"),
              Tab(text: "Product Info"),
            ],
          ),
        ),
        body: TabBarView(
          children: [
            UserInfoPage(user),
            ProductInfoPage(user),
          ],
        ),
      ),
    );
  }
}

class UserInfoPage extends StatefulWidget {
  final User user;

  UserInfoPage(this.user);

  @override
  _UserInfoPageState createState() => _UserInfoPageState();
}

class _UserInfoPageState extends State<UserInfoPage> {
  final TextEditingController fullNameController = TextEditingController();
  final TextEditingController emailController = TextEditingController();
  final TextEditingController newPasswordController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: StreamBuilder<DocumentSnapshot>(
        stream: FirebaseFirestore.instance
            .collection('users')
            .doc(widget.user.uid)
            .snapshots(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) {
            return Center(child: CircularProgressIndicator());
          }

          var userData = snapshot.data!.data() as Map<String, dynamic>;

          fullNameController.text = userData['fullName'];
          emailController.text = userData['email'];

          return Padding(
            padding: const EdgeInsets.all(16.0),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  "Full Name:",
                  style: Theme.of(context).textTheme.bodyText2,
                ),
                TextField(
                  controller: fullNameController,
                  readOnly: true,
                  style: Theme.of(context).textTheme.bodyText2,
                ),
                SizedBox(height: 16),
                Text(
                  "Email:",
                  style: Theme.of(context).textTheme.bodyText2,
                ),
                TextField(
                  controller: emailController,
                  readOnly: true,
                  style: Theme.of(context).textTheme.bodyText2,
                ),
                SizedBox(height: 20),
                ElevatedButton(
                  onPressed: () {
                    // Navigate to the edit page
                    Navigator.push(
                      context,
                      MaterialPageRoute(
                          builder: (context) => EditUserInfoPage(widget.user)),
                    );
                  },
                  child: Text("Edit"),
                ),
                SizedBox(height: 16),
                ElevatedButton(
                  onPressed: () {
                    // Show the dialog to change password
                    showDialog(
                      context: context,
                      builder: (context) => AlertDialog(
                        title: Text("Change Password"),
                        content: Column(
                          mainAxisSize: MainAxisSize.min,
                          children: [
                            TextField(
                              controller: newPasswordController,
                              obscureText: true,
                              decoration: InputDecoration(
                                labelText: "New Password",
                                border: OutlineInputBorder(),
                                fillColor: Colors.white,
                                filled: true,
                              ),
                            ),
                          ],
                        ),
                        actions: [
                          TextButton(
                            onPressed: () {
                              Navigator.pop(context);
                            },
                            child: Text("Cancel"),
                          ),
                          ElevatedButton(
                            onPressed: () async {
                              // Change password
                              await widget.user
                                  .updatePassword(newPasswordController.text);

                              // Close the dialog
                              Navigator.pop(context);
                            },
                            child: Text("Change"),
                          ),
                        ],
                      ),
                    );
                  },
                  child: Text("Change Password"),
                ),
              ],
            ),
          );
        },
      ),
    );
  }
}

class EditUserInfoPage extends StatelessWidget {
  final User user;
  final TextEditingController fullNameController = TextEditingController();
  final TextEditingController emailController = TextEditingController();

  EditUserInfoPage(this.user);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Edit User Information"),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              "Full Name:",
              style: Theme.of(context).textTheme.bodyText2,
            ),
            TextField(
              controller: fullNameController,
              style: Theme.of(context).textTheme.bodyText2,
            ),
            SizedBox(height: 16),
            Text(
              "Email:",
              style: Theme.of(context).textTheme.bodyText2,
            ),
            TextField(
              controller: emailController,
              style: Theme.of(context).textTheme.bodyText2,
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () async {
                // Update user information in Firestore
                await FirebaseFirestore.instance
                    .collection('users')
                    .doc(user.uid)
                    .update({
                  'fullName': fullNameController.text,
                  'email': emailController.text,
                });

                // Navigate back to user info page
                Navigator.pop(context);
              },
              child: Text("Save"),
            ),
          ],
        ),
      ),
    );
  }
}

class ProductInfoPage extends StatefulWidget {
  final User user;

  ProductInfoPage(this.user);

  @override
  _ProductInfoPageState createState() => _ProductInfoPageState();
}

class _ProductInfoPageState extends State<ProductInfoPage> {
  final TextEditingController drinkNameController = TextEditingController();
  final TextEditingController drinkDescriptionController =
      TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              "Add New Drink:",
              style: Theme.of(context).textTheme.bodyText2,
            ),
            TextField(
              controller: drinkNameController,
              decoration: InputDecoration(
                labelText: "Drink Name",
                border: OutlineInputBorder(),
                fillColor: Colors.white,
                filled: true,
              ),
            ),
            SizedBox(height: 16),
            TextField(
              controller: drinkDescriptionController,
              decoration: InputDecoration(
                labelText: "Drink Description",
                border: OutlineInputBorder(),
                fillColor: Colors.white,
                filled: true,
              ),
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () async {
                // Add new drink to Firestore
                await FirebaseFirestore.instance.collection('drinks').add({
                  'name': drinkNameController.text,
                  'description': drinkDescriptionController.text,
                  'userId': widget.user.uid,
                });

                // Clear text fields
                drinkNameController.clear();
                drinkDescriptionController.clear();
              },
              child: Text("Add Drink"),
            ),
            SizedBox(height: 20),
            Text(
              "My Drinks:",
              style: Theme.of(context).textTheme.bodyText2,
            ),
            Expanded(
              child: StreamBuilder<QuerySnapshot>(
                stream: FirebaseFirestore.instance
                    .collection('drinks')
                    .where('userId', isEqualTo: widget.user.uid)
                    .snapshots(),
                builder: (context, snapshot) {
                  if (!snapshot.hasData) {
                    return Center(child: CircularProgressIndicator());
                  }

                  var drinks = snapshot.data!.docs;

                  return ListView.builder(
                    itemCount: drinks.length,
                    itemBuilder: (context, index) {
                      var drink = drinks[index].data() as Map<String, dynamic>;

                      return Card(
                        margin: EdgeInsets.symmetric(vertical: 8),
                        child: ListTile(
                          title: Text(drink['name']),
                          subtitle: Text(drink['description']),
                          trailing: IconButton(
                            icon: Icon(Icons.delete),
                            onPressed: () async {
                              // Delete drink from Firestore
                              await FirebaseFirestore.instance
                                  .collection('drinks')
                                  .doc(drinks[index].id)
                                  .delete();
                            },
                          ),
                        ),
                      );
                    },
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class LoginPage extends StatelessWidget {
  final FirebaseAuthService _authService = FirebaseAuthService();
  final TextEditingController emailController = TextEditingController();
  final TextEditingController passwordController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Login"),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              TextField(
                controller: emailController,
                decoration: InputDecoration(
                  labelText: "Email",
                  border: OutlineInputBorder(),
                  fillColor: Colors.white,
                  filled: true,
                ),
              ),
              SizedBox(height: 16),
              TextField(
                controller: passwordController,
                obscureText: true,
                decoration: InputDecoration(
                  labelText: "Password",
                  border: OutlineInputBorder(),
                  fillColor: Colors.white,
                  filled: true,
                ),
              ),
              SizedBox(height: 20),
              ElevatedButton(
                onPressed: () async {
                  final email = emailController.text;
                  final password = passwordController.text;
                  User? user = await _authService.signInWithEmailAndPassword(
                    email,
                    password,
                  );
                  if (user != null) {
                    Navigator.push(
                      context,
                      MaterialPageRoute(
                        builder: (context) => MyAppInfo(user),
                      ),
                    );
                  } else {
                    // Handle login failure
                    print("Login failed");
                  }
                },
                child: Text("Login"),
              ),
              TextButton(
                onPressed: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(builder: (context) => SignupPage()),
                  );
                },
                child: Text("Don't have an account? Sign up"),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class SignupPage extends StatelessWidget {
  final FirebaseAuthService _authService = FirebaseAuthService();
  final TextEditingController fullNameController = TextEditingController();
  final TextEditingController emailController = TextEditingController();
  final TextEditingController passwordController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Sign Up"),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              TextField(
                controller: fullNameController,
                decoration: InputDecoration(
                  labelText: "Full Name",
                  border: OutlineInputBorder(),
                  fillColor: Colors.white,
                  filled: true,
                ),
              ),
              SizedBox(height: 16),
              TextField(
                controller: emailController,
                decoration: InputDecoration(
                  labelText: "Email",
                  border: OutlineInputBorder(),
                  fillColor: Colors.white,
                  filled: true,
                ),
              ),
              SizedBox(height: 16),
              TextField(
                controller: passwordController,
                obscureText: true,
                decoration: InputDecoration(
                  labelText: "Password",
                  border: OutlineInputBorder(),
                  fillColor: Colors.white,
                  filled: true,
                ),
              ),
              SizedBox(height: 20),
              ElevatedButton(
                onPressed: () async {
                  final fullName = fullNameController.text;
                  final email = emailController.text;
                  final password = passwordController.text;
                  User? user = await _authService.signUpWithEmailAndPassword(
                    email,
                    password,
                    fullName,
                  );

                  if (user != null) {
                    // Save additional user data to Firestore
                    await FirebaseFirestore.instance
                        .collection('users')
                        .doc(user.uid)
                        .set({
                      'fullName': fullName,
                      'email': email,
                      // Add more fields as needed
                    });

                    Navigator.pop(context); // Close the current signup page
                    Navigator.push(
                      context,
                      MaterialPageRoute(builder: (context) => LoginPage()),
                    );
                  } else {
                    // Handle signup failure
                    print("Signup failed");
                  }
                },
                child: Text("Sign Up"),
              ),
              TextButton(
                onPressed: () {
                  Navigator.pop(context);
                },
                child: Text("Already have an account? Log in"),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class FirebaseAuthService {
  final FirebaseAuth _auth = FirebaseAuth.instance;

  Future<User?> signInWithEmailAndPassword(
      String email, String password) async {
    try {
      UserCredential authResult = await _auth.signInWithEmailAndPassword(
        email: email,
        password: password,
      );
      User? user = authResult.user;
      return user;
    } catch (e) {
      print("Login error: $e");
      return null;
    }
  }

  Future<User?> signUpWithEmailAndPassword(
      String email, String password, String fullName) async {
    try {
      UserCredential authResult = await _auth.createUserWithEmailAndPassword(
        email: email,
        password: password,
      );

      User? user = authResult.user;
      return user;
    } catch (e) {
      print("Signup error: $e");
      return null;
    }
  }
}

