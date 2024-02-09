<script setup lang="ts">
/**
 * Import VueJS
 *
 */
import { onMounted, ref } from 'vue';

/**
 * Import PeerJS
 *
 */
import { Peer } from 'peerjs';

/**
 * peer id
 * You and the other one connected to you
 *
 */
const lastPeerId = ref(null);
const otherId = ref('');

/**
 * Peer connection between you and other
 *
 */
const peer: Peer = ref(null);
const conn: any = ref(null);

/**
 * Status of the connection
 *
 */
const status = ref('Waiting connection...');

/**
 * Messages sent and received
 *
 */
const messages: any = ref(['Welcome to PeerJS Chat!']);
const message = ref('');

/**
 * Connect to the other peer
 *
 */
const connect = () => {
  /**
   * Close old connection if exist
   *
   */
  if (conn.value) conn.value.close();

  /**
   * Create connection to destination
   * peer specified in the input field
   *
   */
  conn.value = peer.value.connect(otherId.value, {
    reliable: true
  });

  /**
   * Update status label when connected
   *
   */
  conn.value.on('open', function () {
    status.value = `Connected to: ${conn.value.peer}`;
    console.log(`Connected to: ${conn.value.peer}`);
  });

  ready();
};

/**
 * Call when connection ready
 * called when connection is ready when you are click connect to toher peer
 * or when other peer is connected to you
 *
 */
const ready = () => {
  /**
   * Handle incoming data
   * (messages only since this is the signal sender)
   *
   */
  conn.value.on('data', function (data: string) {
    messages.value.push(`<span>Other:</span> ${data}`);
  });
  conn.value.on('close', function () {
    status.value = 'Connection closed';
  });
};

/**
 * Send message to other peer
 *
 */
const sendMessage = () => {
  if (conn.value && conn.value.open) {
    conn.value.send(message.value);
    messages.value.push(`<span>You:</span> ${message.value}`);
    console.log(`Message sent: ${message.value}`);

    /**
     * Clear input field
     *
     */
    message.value = '';
  } else {
    console.log('Connection is closed');
  }
}

onMounted(() => {
  /**
   * Initialize PeerJS
   */
  peer.value = new Peer({ debug: 2 });

  /**
   * Handle open event
   *
   */
  peer.value.on('open', (id: string) => {
    if (peer.value.id === null) {
      console.log('Received null id from peer open');
      peer.value.id = lastPeerId.value;
    } else {
      lastPeerId.value = peer.value.id;
    }
    console.log('ID: ' + peer.value.id);
  });

  /**
   * Handle incoming connection
   * This is called when you connected to other peer
   *
   */
  peer.value.on('connection', (c: any) => {
    if (conn.value && conn.value.open) {
      c.on('open', () => {
          c.send('Already connected to another client');
          setTimeout(function() { c.close(); }, 500);
      });
      return;
    }
    conn.value = c;
    console.log(`Connected to: ${conn.value.peer}`);
    status.value = 'Connected';

    /**
     * Call ready event to handle incoming data
     *
     */
    ready();
  });

  peer.value.on('disconnected', () => {
    status.value = 'Connection lost. Please reconnect';
    console.log('Connection lost. Please reconnect');

    // Workaround for peer.value.reconnect deleting previous id
    peer.value.id = lastPeerId.value;
    peer.value._lastServerId = lastPeerId.value;
    peer.value.reconnect();
  });
  peer.value.on('close', () => {
    conn.value = null;
    status.value = 'Connection destroyed. Please refresh';
    console.log('Connection destroyed');
  });
  peer.value.on('error', (err: any) => {
    console.log(err);
    status.value = err.message;
  });
})
</script>

<template>
  <main>
    <div>Your ID: <b>{{lastPeerId}}</b></div>
    <div>Status: <b>{{status}}</b></div>
    <br />
    <hr/>
    <input type="text" placeholder="Other peer ID..." v-model="otherId">
    <button v-on:click="connect">Connect</button>
    <br />
    <br />
    <hr/>
    <div class="messages">
      <div class="message" v-for="(message, i) in messages" :key="i" v-html="message"></div>
    </div>
    <hr/>
    <input type="text" placeholder="Type chat..." v-model="message">
    <button v-on:click="sendMessage">Send</button>
  </main>
</template>
