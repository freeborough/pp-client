<script>
  import { onMount, onDestroy } from "svelte";
  import { Peer } from "peerjs";

  let stdout = "";
  function log(message) {
    stdout = `${stdout}\n${message}`;
  }

  log("WebRTC Client");

  // Detrmine our identity based on our window position.
  let ourId;
  let theirId;
  let message;

  const idOne = "freeborough-one";
  const idTwo = "freeborough-two";

  if (window.screenX > 5000) {
    ourId = idTwo;
    theirId = idOne;
  } else {
    ourId = idOne;
    theirId = idTwo;
  }

  var peer;
  var dataConnection;

  onMount(() => {
    log(`Creating peer: ${ourId}`);
    peer = new Peer(ourId, { debug: 2 });

    peer.on("connection", (conn) => {
      log(`peer.on.connection`);
      conn.on("data", (data) => {
        log(`peer.on.connection.on.data:`);
        log(data);
      });

      conn.on("open", () => {
        log(`peer.on.connection.on.open`);
        getConnection();
      });
    });

    peer.on("open", () => {
      log(`peer.on.open`);
      getConnection();
    });

    peer.on("error", (err) => {
      log(`ERROR: ${JSON.stringify(err)}`);
    });

    getConnection();
  });

  onDestroy(() => {
    disconnect();
  });

  function connect() {
    log(`Connecting to ${theirId}`);
    dataConnection = peer.connect(theirId);
  }

  function disconnect() {
    log(`Disconnecting.`);
    dataConnection.close();
    peer.disconnect();
  }

  function getConnection() {
    if (dataConnection && dataConnection.open) {
      log(`Re-using open connection.`);
    } else {
      log(`Creating new connection.`);
      dataConnection = peer.connect(theirId);
    }

    return dataConnection;
  }

  function send(msg) {
    log(`send: ${msg}`);
    dataConnection.send(msg);
  }

  function sendMessage() {
    send(message);
  }

</script>

<h1>WebRTC Client</h1>

<div class="field">
  Our ID: {ourId}
</div>

<div class="field">
  Their ID: {theirId}
</div>

<button on:click={connect}>Connect</button>
<input type="text" bind:value={message} />
<button on:click={sendMessage}>Send</button>

<pre>{stdout}</pre>