import 'package:flutter/material.dart';
import 'package:geolocator/geolocator.dart';
import 'package:shared_preferences/shared_preferences.dart';
import 'package:shake/shake.dart';
import 'contacts_screen.dart';

void main() {
  runApp(const ShieldHerApp());
}

class ShieldHerApp extends StatelessWidget {
  const ShieldHerApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'ShieldHer',
      home: const HomeScreen(),
    );
  }
}

class HomeScreen extends StatefulWidget {
  const HomeScreen({super.key});

  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  String locationText = "Location not fetched";

  Future<void> getLocation() async {
    LocationPermission permission;

    permission = await Geolocator.checkPermission();

    if (permission == LocationPermission.denied) {
      permission = await Geolocator.requestPermission();
    }

    Position position = await Geolocator.getCurrentPosition(
      desiredAccuracy: LocationAccuracy.high,
    );

    setState(() {
      locationText =
      "Lat: ${position.latitude}\nLng: ${position.longitude}";
    });
  }

  Future<void> smartSOS() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();

    List<String> contacts =
        prefs.getStringList('contacts') ?? [];

    Position position = await Geolocator.getCurrentPosition(
      desiredAccuracy: LocationAccuracy.high,
    );

    String message =
        "EMERGENCY ALERT!\n\n"
        "Location:\n"
        "Lat: ${position.latitude}\n"
        "Lng: ${position.longitude}\n\n"
        "Emergency Contacts:\n";

    if (contacts.isEmpty) {
      message += "No contacts added";
    } else {
      for (int i = 0; i < contacts.length; i++) {
        message += "${i + 1}. ${contacts[i]}\n";
      }
    }

    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text("🚨 SOS ALERT"),
        content: SingleChildScrollView(
          child: Text(message),
        ),
        actions: [
          TextButton(
            onPressed: () {
              Navigator.pop(context);
            },
            child: const Text("OK"),
          ),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("ShieldHer"),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                smartSOS();
              },
              child: const Text("EMERGENCY SOS"),
            ),

            const SizedBox(height: 20),

            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) =>
                    const ContactsScreen(),
                  ),
                );
              },
              child: const Text("Emergency Contacts"),
            ),

            const SizedBox(height: 20),

            ElevatedButton(
              onPressed: getLocation,
              child: const Text("Get My Location"),
            ),

            const SizedBox(height: 20),

            Text(
              locationText,
              textAlign: TextAlign.center,
            ),
          ],
        ),
      ),
    );
  }
}