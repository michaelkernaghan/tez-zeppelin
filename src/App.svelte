<script lang="ts">
  import { onMount, onDestroy } from "svelte";
  import { TezosToolkit } from "@taquito/taquito";
  import { BeaconWallet } from "@taquito/beacon-wallet";
  import { NetworkType } from "@airgap/beacon-sdk";
  import { HubConnectionBuilder, HubConnection } from "@microsoft/signalr";

  let Tezos: TezosToolkit;
  let wallet: BeaconWallet;
  let subscription: HubConnection;
  let blockHead: { protocol: string; level: number; lastUpdate: string };
  let confirmed: { confirmation: string};
  let title_list = [
  {
      "subtitle": "Stairway to Hodl",
  },
  {
      "subtitle": "Doged and Confused",
  },
  {
      "quote": "... waiting for the angels of Avalon, waiting for the eastern glow.",
  },
  {
      "quote": "They talk of days for which they sit and wait. All will be revealed.",
  },
]

  const rpcUrl = "https://api.tez.ie/rpc/mainnet";
  const packages: { name: string; display: string; version: number }[] = [
    { name: "svelte", display: "Svelte", version: 3 },
    { name: "webpack", display: "Webpack", version: 5 },
    { name: "typescript", display: "TypeScript", version: 3 },
    { name: "taquito", display: "Taquito", version: 8 },
    { name: "beacon", display: "Beacon SDK", version: 2 }
  ];

  const connect = async () => {
    try {
      wallet = new BeaconWallet({
        name: "Tez Zeppelin Wallet",
        preferredNetwork: NetworkType.MAINNET
      });
      await wallet.requestPermissions({
        network: {
          type: NetworkType.MAINNET,
          rpcUrl
        }
      });
      Tezos.setWalletProvider(wallet);
    } catch (err) {
      console.error(err);
    }
  };

  const disconnect = () => {
    wallet.client.destroy();
    wallet = undefined;
  };

  let success = false;
  let loading = false;
  const transfer = async () => {
    loading = true
    const amount = 0.1;
    const address = 'tz1h1LzP7U8bNNhow8Mt1TNMxb91AjG3p6KH';
    const op = await Tezos.wallet.transfer({ to: address, amount: amount }).send();
    await op.confirmation();
    success = true;
    loading = false
  }   


  const subscribeToEvents = async () => {
    const connection = new HubConnectionBuilder()
      .withUrl("https://api.tzkt.io/v1/events")
      .build();

    // auto-reconnect
    connection.onclose(subscribeToEvents);

    connection.on("head", msg => {
      blockHead = {
        protocol: msg.data.protocol,
        level: msg.data.level,
        lastUpdate: msg.data.timestamp
      };
    });

    // open connection
    await connection.start();
    // subscribe to head
    await connection.invoke("SubscribeToHead");
    // return connection instance
    return connection;
  };

  onMount(async () => {
    Tezos = new TezosToolkit(rpcUrl);
    const headerInfo = await Tezos.rpc.getBlockHeader();
    blockHead = {
      protocol: headerInfo.protocol,
      level: headerInfo.level,
      lastUpdate: headerInfo.timestamp
    };
    subscription = await subscribeToEvents();
  });

  onDestroy(async () => {
    // closes subscription when component unmounts
    await subscription.stop();
  });
</script>

<style lang="scss">
  $tezos-blue: #050505;

  .container {
    font-size: 20px;
    max-width: 80%;

    .subtitle {
      font-size: 25px;
      font-family: cursive;
      font-style: oblique;
      color: #333;
      margin: 10px;
    }

    .chain-info {
      font-size: 20px;

      p {
        margin: 5px;
        font-style: italic;
        color: $tezos-blue;
      }
    }

    button {
      appearance: none;
      border: solid 2px $tezos-blue;
      border-radius: 5px;
      background-color: white;
      padding: 20px;
      font-size: 20px;
      font-family: cursive;
      font-style: oblique;
      color: $tezos-blue;
      transition: 0.3s;
      cursor: pointer;
      outline: none;

      &:hover {
        color: white;
        background-color: $tezos-blue;
      }
    }
  }
</style>

<main>
  <div class="container">
    <div class="subtitle">Doged and Confused</div>
    <br />
    <br />
    <div>
      <img src={'images/tezoso.png'} alt="Tezoso" width="600" height="300" />
      <br />
    </div>
    <br />
    <br />
    <div>
      {#if wallet}
        <div>
          {#if loading}
          <br />
          <br />
            <img src={'images/spinning_arrows.gif'} alt="loading...">
            <br />
            <div class="subtitle">  
              <br/>      
              <p>They talk of days for which they sit and wait. All will be revealed.</p>
            </div>
          {:else}
          <br />
          <br />
            <button on:click={transfer}>Next, touch here to experience a Tez Zeppelin NFT for 0.1 Tez</button>  
            <p></p>
            {#if success} 
            <button on:click={disconnect}>Close the wallet by touching here</button>
            <br />
            <div class="subtitle">        
             <p>robertplaintext.tezzeppelin.tez</p>
            <p>jimmypagefault.tezzeppelin.tez</p>
            <p>johnpauljavascript.tezzeppelin.tez</p>
            <p>johnbotnet.tezzeppelin.tez</p>
            </div>
            <br />
            
            <img src={'images/tez-zeppelin-cover.png'} alt="Jimmy Pagefault says thanks!">
            <br />
            {/if}
          {/if} 
          </div>
      {:else}
        <button on:click={connect}>First, touch here to open a wallet and get 0.1 tez</button>
        <br />
        <br />
      {/if}
    </div>   
    <br />
    <br />
  </div>
</main>
