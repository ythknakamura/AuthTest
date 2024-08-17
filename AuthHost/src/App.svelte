<script lang="ts">
  import { onDestroy, onMount } from "svelte";
  import { createClient } from "@supabase/supabase-js";

  const email = "admin@kenryo.ed.jp";
  const password = "ma=f";
  let responseMsg = "";

  const supabase = createClient('https://mcvfbonxuvlnxbdskmnw.supabase.co', 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im1jdmZib254dXZsbnhiZHNrbW53Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3MjM4MDM0MjksImV4cCI6MjAzOTM3OTQyOX0.BF5r5wACNjvlmbaFc0SQ54EvEzQEDLLE4cu41jH_EAY');

  onMount(async ()=>{
    await supabase.auth.signInWithPassword({email,password});
    showMessage("データベースの変更の監視開始");
    
    supabase.channel("admin")
      .on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'submits' }, payload => {
        const {new:{uid}} = payload;
        showMessage(`データベース変更検知 : ${uid}`);
        replayMessage(uid);
      }).subscribe();
  });

  onDestroy(() => {
    supabase.removeAllChannels();
    supabase.auth.signOut();
  });

  function showMessage(message:string):void{
    const date = new Date().toLocaleString("ja-JP",{
      hour:"2-digit",
      minute:"2-digit",
      fractionalSecondDigits:2});
    responseMsg += `[${date}] ${message}\n`;
  }

  function replayMessage(uid:string):void{
    const channel =  supabase.channel(uid);
    // ここでuidから資格情報を調べて、解錠するかどうかを判断する
    channel.subscribe((status) => {
      if (status === 'SUBSCRIBED') {
        showMessage(`解錠メッセージを送信 : ${uid}`);
        channel.send({
          type: 'broadcast',
          event: 'unlock',
          payload: `ホスト >>> クライアント: 解錠したよ！`,
        });
      }
    });
  }
</script>

<svelte:head>
  <title>Auth Host</title>
</svelte:head>

<main>  
  <h1>Auth Host</h1>
  <div>
    <div>メッセージ</div>
    <div id="response">{responseMsg}</div>
  </div>
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1ex;
  }

  main>div{
    display: grid;
    grid-template-columns: 50em; 
    column-gap :3em;
    padding: 1em;
    background: lightpink;
  }

  #response{
    background: white;;
    border: 1px solid black;
    min-height: 10ex;
    overflow-y: scroll;
    white-space: pre-wrap;
  }
</style>