<template>
   <div class="wrapper">
        <div class="header">
            🗨️ WebChat
        </div>

        <div class="message-section" id="messageSection">
        </div>

        <div class="input-section">
            <input type="text" class="input" id="txtMessage"/>
            <button class="btn" id="btnSend">Send</button>
        </div>
    </div>
</template>
<script setup>
import * as signalR from "@microsoft/signalr";
const connection = new signalR.HubConnectionBuilder()
    .withUrl("https://localhost:7162/chathub")
    .configureLogging(signalR.LogLevel.Information)
    .build();

const start = async () => {
    try {
        await connection.start();
        console.log("Connected to signal r hub");
    } catch (error) {
        console.log(error);
    }
}

// getting the username from prompt modal and store in session storage
// session storage, exists till a browser tab is open
const joinUser = async () => {
    const name = window.prompt('Enter the name: ');
    if (name)
    {
        sessionStorage.setItem('user', name);   
        // here user will join the chat
        await joinChat(name);
    }
}

const joinChat = async (user) => {
    if (!user)
       return;
    try {
        const message = `${user} joined`;
        await connection.invoke("JoinChat", user, message);
    } catch (error) {
        console.log(error);
    }
}

// fetching the user from sesstion storage
const getUser = () => sessionStorage.getItem('user')
// method for getting notified by server
const receiveMessage= async () => {
    const currentUser = getUser();
    if (!currentUser)
        return;
    try {
        await connection.on("ReceiveMessage", (user, message) => {
         const messageClass = currentUser === user ? "send" : "received";
            appendMessage(message, messageClass);
            const alertSound = new Audio('chat-sound.mp3');
            alertSound.play();
       })
    } catch (error) {
        console.log(error);
    }
}
    // append meessage to the message-section
    const appendMessage = (message,messageClass) => {
    const messageSectionEl = document.getElementById('messageSection');
    const msgBoxEl = document.createElement("div");
    msgBoxEl.classList.add("msg-box");
    msgBoxEl.classList.add(messageClass);
    msgBoxEl.innerHTML = message;
    messageSectionEl.appendChild(msgBoxEl);
}



const sendMessage = async (user,message) => {
    
    try {
        await connection.invoke('SendMessage', user, message);
    } catch (error) {
        console.log(error);
    }
}


// starting the app
const startApp = async () => {
    await start(); // connection will stablised
    await joinUser();
    await receiveMessage();
}

onMounted(() => {
// binding the event for send button
document.getElementById('btnSend').addEventListener('click', async (e) => {
    e.preventDefault();
    const user = getUser();
    if (!user)
        return;
    const txtMessageEl = document.getElementById('txtMessage');
    const msg = txtMessageEl.value;
    if (msg) {
        // call the sendmessage api
        await sendMessage(user,`${user}: ${msg}`);  // john: hey guys
        txtMessageEl.value = "";
    }
})
startApp();
})
</script>
<style >
*{
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

::-webkit-scrollbar {
    width: 10px;
  }

  ::-webkit-scrollbar-track {
    box-shadow: inset 0 0 5px grey; 
    border-radius: 10px;
  }

  ::-webkit-scrollbar-thumb {
    background: rgb(135, 1, 145); 
    border-radius: 10px;
  }
  
  ::-webkit-scrollbar-thumb:hover {
    background: rgb(162, 2, 170); ; 
  }


.wrapper{
    width: 100%;
    height: 100vh;
    background: rgb(2,253,245);
    background: linear-gradient(90deg, rgba(2,253,245,1) 19%, rgba(252,134,255,1) 100%);
    font-family: 'Poppins', sans-serif;

}

.header{
    font-size: 28px;
    text-align: center;
    padding: 15px 0px;
}

.message-section{
    height: 400px;
    width: 90%;
    margin: auto;
    border: 1px solid;
    border-radius: 10px;
    overflow: auto;
}

.msg-box{
    padding: 10px;
    width: 35%;
    border: 1px solid black;
    font-size: 20px;
    margin: 20px 15px;
    border-radius: 5px;
    clear: both;
    word-break: break-all;
    box-shadow: 5px 5px #8b8b8b;
}

.received{
  float: left;
  background-color: rgb(255, 252, 214);
}

.send{
  float: right;
  background-color: rgb(214, 255, 214);
}

/* input section */
.input-section{
    border: 1px solid black;
    height: 60px;
    width: 90%;
    margin: auto;
    margin-top: 10px;
    border-radius: 10px;
    display: flex;
    justify-content: center;
    align-items: center;
}

.input{
    width: 90%;
    padding:10px;
    font-size: 20px;
    border: 1px solid #969696;
    border-radius: 10px;
    margin:0px 5px ;
}

.input:focus{
    border: 2px solid #757575;
    outline: none;
}

.btn{
    font-size: 20px;
    padding: 10px 15px;
    border: none;
    border-radius: 10px;
    cursor: pointer;
    background: rgb(98, 0, 98);
    color: white;
    margin:0px 5px ;
}

.btn:hover{
    background: rgb(69, 0, 69);
}
</style>