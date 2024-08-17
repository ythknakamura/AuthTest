<script lang="ts">
  import { onDestroy, onMount } from "svelte";
  import { createClient } from "@supabase/supabase-js";

  let isLogin :boolean;
  let email = "test@kenryo.ed.jp";
  let password = "ma=f";
  let loginMsg = "";
  let submitMsg = "";
  let responseMsg = "";
  const supabase = createClient('https://mcvfbonxuvlnxbdskmnw.supabase.co', 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im1jdmZib254dXZsbnhiZHNrbW53Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3MjM4MDM0MjksImV4cCI6MjAzOTM3OTQyOX0.BF5r5wACNjvlmbaFc0SQ54EvEzQEDLLE4cu41jH_EAY');

  onMount(()=>{
    getUserIDAsync();
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

  async function getUserIDAsync() : Promise<string>{
    const {data : {user}} = await supabase.auth.getUser();
    isLogin = user !== null;
    if(isLogin){
      loginMsg = `ログイン成功 : ${user.id}`;
      return user.id;
    }
    else{
      loginMsg = "ログアウト中";
      supabase.removeAllChannels();
      return null;
    }
  }

  async function loginButtonClickedAsync(): Promise<void>{
    const uid = await getUserIDAsync();
    if(uid){
      supabase.auth.signOut();
    }
    else{ 
      const { error } = await supabase.auth.signInWithPassword({email,password});
      if(error){
        console.log(error.message);
      }
    }
    await getUserIDAsync();
  }

  async function sumbitButtonClickedAsync(): Promise<void>{
    const uid = await getUserIDAsync();
    const channel = supabase.channel(uid);
      channel.on("broadcast", { event: "unlock" }, ({payload}) => {
        showMessage(payload);
        channel.unsubscribe();
      }).subscribe();

    const { data, error } = await supabase.from("submits").insert({});
    if(error){
      console.log(data);
      submitMsg = error.message;
    }
    else{
      submitMsg = `サブミット成功`;
      showMessage("クライアント >>> ホスト : 解錠依頼中");
    }
  }

</script>

<svelte:head>
  <title>Auth Client</title>
</svelte:head>


<main>  
  <h1>Auth Client</h1>
  <div>
    <label for="email">メールアドレス</label>
    <input id="email" type="text" bind:value={email}/>
    <div>@kenryo.ed.jp以外は受け付けない</div>

    <label for="password">パスワード</label>
    <input id="password" type="password" bind:value={password}/>
    <div>「ma=f」です</div>

    <button on:click={loginButtonClickedAsync}>{isLogin?"ログアウト":"ログイン"}</button>
    <div style="grid-column:2/4;">{loginMsg}</div>
    
    <button on:click={sumbitButtonClickedAsync}>サブミット</button>
    <div style="grid-column:2/4;">{submitMsg}</div>
    
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
    grid-template-columns: 10em 1fr 1fr; 
    grid-template-rows: 3ex 3ex 3ex 3ex auto;
    column-gap :3em;
    row-gap: 1em;
    padding: 1em;
    background: lightgray;
  }

  #response {
    background: white;;
    border: 1px solid black;
    grid-column:1/4;
    min-height: 10ex;
    overflow-y: scroll;
    white-space: pre-wrap;
  }
</style>