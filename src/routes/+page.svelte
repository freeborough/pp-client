<script>
  import { onMount, onDestroy } from "svelte";
  import { Peer } from "peerjs";

  let stdout = "";
  let logging = false;

  function log(message) {
    if (logging) {
      stdout = `${stdout}\n${message}`;
    }
  }

  log("WebRTC Client");

  // Detrmine our identity based on our window position.
  let ourId;
  let theirId;
  let messages = [];

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
        messages = [`${theirId}: ${data}`, ...messages];
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

    document.getElementById("toSend").focus();
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
    messages = [`${ourId}: ${msg}`, ...messages];
    dataConnection.send(msg);
  }

  function sendMessage() {
    const toSend = document.getElementById("toSend");
    send(`${toSend.value}`);
    toSend.value = "";
    toSend.focus();
  }

  function onKeyPress(e) {
    if (e.code == "Enter") {
      sendMessage();
    }
  }

</script>

<h1>WebRTC Client</h1>

<div class="field">
  Our ID: {ourId}
</div>

<div class="field">
  Their ID: {theirId}
</div>

<input type="text" id="toSend" on:keypress={onKeyPress} />
<button on:click={sendMessage}>Send</button>

<div class="messages">
  {#if messages.length > 0}
    {#each messages as message}
      <div class="message">{message}</div>
    {/each}
  {/if}
</div>

<pre>{stdout}</pre>