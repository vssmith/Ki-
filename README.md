# Ki-# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  http: ^0.13.4
  path_provider: ^2.0.11
  audioplayers: ^1.0.1
  flutter_sound: ^9.2.13
  permission_handler: ^10.2.0
// lib/main.dart
import 'package:flutter/material.dart';
import 'screens/record_screen.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'EchoTale',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: RecordScreen(),
    );
  }
}
// lib/screens/record_screen.dart
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:flutter_sound/flutter_sound.dart';
import 'package:path_provider/path_provider.dart';
import 'package:http/http.dart' as http;
import 'package:permission_handler/permission_handler.dart';
import '../services/api_service.dart';

class RecordScreen extends StatefulWidget {
  @override
  _RecordScreenState createState() => _RecordScreenState();
}

class _RecordScreenState extends State<RecordScreen> {
  FlutterSoundRecorder? _recorder;
  bool isRecording = false;
  String? filePath;

  @override
  void initState() {
    super.initState();
    _recorder = FlutterSoundRecorder();
    _initRecorder();
  }

  Future<void> _initRecorder() async {
    await _recorder!.openRecorder();
    await Permission.microphone.request();
  }

  Future<void> _startRecording() async {
    final dir = await getApplicationDocumentsDirectory();
    filePath = '${dir.path}/recorded_audio.aac';
    await _recorder!.startRecorder(toFile: filePath);
    setState(() => isRecording = true);
  }

  Future<void> _stopRecording() async {
    await _recorder!.stopRecorder();
    setState(() => isRecording = false);
  }

  Future<void> _uploadAudio() async {
    if (filePath == null) return;
    String? response = await ApiService.uploadAudio(filePath!);
    if (response != null) {
      print("Upload erfolgreich: $response");
    } else {
      print("Fehler beim Hochladen");
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Stimme aufnehmen")),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              child: Text(isRecording ? "Stoppen" : "Aufnehmen"),
              onPressed: isRecording ? _stopRecording : _startRecording,
            ),
            ElevatedButton(
              child: Text("Hochladen"),
              onPressed: _uploadAudio,
            ),
          ],
        ),
      ),
    );
  }

  @override
  void dispose() {
    _recorder!.closeRecorder();
    super.dispose();
  }
}
// backend/server.js
const express = require("express");
const multer = require("multer");
const cors = require("cors");
const dotenv = require("dotenv");
const axios = require("axios");

dotenv.config();
const app = express();
app.use(cors());
app.use(express.json());

const upload = multer({ dest: "uploads/" });

app.post("/upload", upload.single("audio"), (req, res) => {
    console.log("Datei erhalten:", req.file.path);
    res.json({ message: "Upload erfolgreich", filePath: req.file.path });
});

app.post("/generate-audio", async (req, res) => {
    try {
        const { text } = req.body;
        const response = await axios.post(
            "https://api.elevenlabs.io/v1/text-to-speech",
            {
                api_key: process.env.ELEVENLABS_API_KEY,
                voice: "dein-trainiertes-modell",
                text: text
            }
        );
        res.json({ audioUrl: response.data.audio_url });
    } catch (error) {
        console.error("Fehler:", error);
        res.status(500).json({ error: "Fehler beim Generieren der Stimme" });
    }
});

app.listen(5000, () => console.log("🚀 Server läuft auf Port 5000"));
# .gitignore
node_modules/
uploads/
.env
build/
android/.gradle/
ios/Pods/
ios/.symlinks/
# GitHub hochladen
git init
git add .
git commit -m "Erster Commit"
git branch -M main
git remote add origin https://github.com/vssmith/Ki-.git
git push -u origin main