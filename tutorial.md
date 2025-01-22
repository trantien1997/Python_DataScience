
# Firebase WebRTC Video Chat

A simple video chat application built using Firebase and WebRTC.

---

## Prerequisites

Ensure you have the following installed:
- **Node.js** (LTS version is recommended).
- **npm** (comes with Node.js).
- A Google account to set up Firebase.
- A modern web browser that supports WebRTC (e.g., Chrome, Firefox).

---

## Steps to Build the Application

### 1. Set Up a Firebase Project
1. **Go to Firebase Console**:  
   Open [Firebase Console](https://console.firebase.google.com/) and create a new project.

2. **Enable Firestore**:
   - Navigate to the **Firestore Database** section.
   - Click **Create Database** and choose ** Start in test mode**.
   - Complete the setup process.

3. **Enable Firebase Hosting**:
   - Go to the **Hosting** section.
   - Follow the instructions to set up Firebase Hosting.

---

### 2. Create a Basic Web Application

#### Initialize a New Project
1. Open your terminal and create a folder for your project:
   ```bash
   mkdir firebase-webrtc
   cd firebase-webrtc
   npm init -y
   ```

2. Install Firebase SDK:
   ```bash
   npm install firebase
   ```

3. Set Up Firebase Configuration:
   - In your Firebase Console, go to **Project Settings** > **General** > **Your apps** > **Add app** (choose Web).
   - Copy the Firebase configuration. Example:
     ```javascript
     const firebaseConfig = {
       apiKey: "YOUR_API_KEY",
       authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
       projectId: "YOUR_PROJECT_ID",
       storageBucket: "YOUR_PROJECT_ID.appspot.com",
       messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
       appId: "YOUR_APP_ID"
     };
     ```

---

### 3. Implement the Frontend

#### Create HTML and JavaScript Files

1. **index.html**:
   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <title>Firebase WebRTC</title>
   </head>
   <body>
     <h1>WebRTC Video Chat</h1>
     <video id="localVideo" autoplay playsinline></video>
     <video id="remoteVideo" autoplay playsinline></video>
     <button id="startButton">Start Call</button>
     <button id="hangupButton">Hang Up</button>
     <script src="app.js"></script>
   </body>
   </html>
   ```

2. **app.js**:
   ```javascript
   import { initializeApp } from 'firebase/app';
   import { getFirestore, doc, setDoc, onSnapshot } from 'firebase/firestore';

   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
     projectId: "YOUR_PROJECT_ID",
     storageBucket: "YOUR_PROJECT_ID.appspot.com",
     messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
     appId: "YOUR_APP_ID"
   };

   const app = initializeApp(firebaseConfig);
   const db = getFirestore(app);

   const servers = {
     iceServers: [
       { urls: 'stun:stun.l.google.com:19302' },
     ],
     iceCandidatePoolSize: 10,
   };

   let pc = new RTCPeerConnection(servers);
   let localStream = null;
   let remoteStream = new MediaStream();

   const localVideo = document.getElementById('localVideo');
   const remoteVideo = document.getElementById('remoteVideo');

   document.getElementById('startButton').addEventListener('click', startCall);
   document.getElementById('hangupButton').addEventListener('click', hangUp);

   async function startCall() {
     localStream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
     localStream.getTracks().forEach((track) => pc.addTrack(track, localStream));
     localVideo.srcObject = localStream;

     const callDoc = doc(db, 'calls', 'sampleCall');
     const offerCandidates = callDoc.collection('offerCandidates');
     const answerCandidates = callDoc.collection('answerCandidates');

     pc.onicecandidate = (event) => {
       event.candidate && setDoc(offerCandidates.doc(), event.candidate.toJSON());
     };

     pc.ontrack = (event) => {
       event.streams[0].getTracks().forEach((track) => {
         remoteStream.addTrack(track);
       });
       remoteVideo.srcObject = remoteStream;
     };

     const offerDescription = await pc.createOffer();
     await pc.setLocalDescription(offerDescription);

     const offer = {
       sdp: offerDescription.sdp,
       type: offerDescription.type,
     };

     await setDoc(callDoc, { offer });

     onSnapshot(callDoc, (snapshot) => {
       const data = snapshot.data();
       if (!pc.currentRemoteDescription && data?.answer) {
         const answerDescription = new RTCSessionDescription(data.answer);
         pc.setRemoteDescription(answerDescription);
       }
     });

     onSnapshot(answerCandidates, (snapshot) => {
       snapshot.docChanges().forEach((change) => {
         if (change.type === 'added') {
           const candidate = new RTCIceCandidate(change.doc.data());
           pc.addIceCandidate(candidate);
         }
       });
     });
   }

   async function hangUp() {
     pc.close();
     pc = null;
     localStream.getTracks().forEach((track) => track.stop());
   }
   ```

---

### 4. Deploy with Firebase Hosting

1. **Initialize Firebase Hosting**:
   ```bash
   firebase login
   firebase init hosting
   ```
   - Select the Firebase project you created earlier.
   - Set the public directory to the folder containing your `index.html` (e.g., `firebase-webrtc`).

2. **Deploy Your Application**:
   ```bash
   firebase deploy
   ```

3. **Access Your Application**:
   After deployment, Firebase will provide a hosting URL. Open it in your browser to test the video chat app.

---

### 5. Test Your Application
1. Open the app in two different browsers or tabs.
2. Start a call in one tab, and accept it in the other.
3. You should see the video streams appear on both tabs.

---

## Future Enhancements

This is a basic WebRTC video chat app. You can extend it further by adding:
- Authentication with Firebase Auth.
- Better UI/UX with frameworks like React or Angular.
- Additional signaling mechanisms.
