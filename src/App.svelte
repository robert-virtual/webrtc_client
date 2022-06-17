<script lang="ts">
  import "./app.css";
  import { io } from "socket.io-client";
  const socket = io("ws://localhost:5000/");
  let mainRoom;
  let roomId: string;

  const peerConnection = new RTCPeerConnection();
  peerConnection.createOffer().then(async (offer) => {
    await peerConnection.setLocalDescription(offer);
    const roomWithOffer = {
      offerType: offer.type,
      offerSdp: offer.sdp,
    };
    // insert room
    socket.emit("create-room", roomWithOffer);
    socket.on("room-created", (roomCreated) => {
      mainRoom = roomCreated;
    });
  });

  socket.on("room-updated", async (room) => {
    console.log("room updated");
    mainRoom = room;
    if (!peerConnection.currentRemoteDescription && room.answer) {
      console.log("set remote description");
      const answer = new RTCSessionDescription(room.answer);
      await peerConnection.setRemoteDescription(answer);
    }
  });
  let allRooms = [];
  socket.on("rooms", (rooms) => {
    console.log("rooms: ",rooms)
    allRooms = rooms;
  });
  const joinRoom = async () => {
    const room = allRooms.find((r) => r.id == roomId);
    if (!room) {
      alert("Room not found");
      return;
    }
    const offer = { type: room.offerType, sdp: room.offerSdp };
    peerConnection.setRemoteDescription(offer);
    const answer = await peerConnection.createAnswer();
    await peerConnection.setLocalDescription(answer);
    socket.emit("create-answer", {
      id: roomId,
      type: answer.type,
      sdp: answer.sdp,
    });
    console.log("answer created");
  };
</script>

<main>
  <h2>Web Rtc</h2>
  <input type="text" bind:value={roomId} placeholder="Room Id" />
  <button on:click={joinRoom}>join room</button>
  <pre>
  {JSON.stringify(mainRoom, null, 2)}
  </pre>
  <p>Rooms:</p>
  <pre>
  {JSON.stringify(allRooms, null, 2)}
  </pre>
</main>

<style>
</style>
