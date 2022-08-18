<template>

<div>

    <Push width="300">
      <h3>ISCP GoldChain</h3>
 
      <a id="instruct" href="#" @click="showHome()">
        <span>Instruction</span>
      </a>
      <a id="vote" href="#" @click="showVote()">
        <span>Vote</span>
      </a>
      <a id="result" href="#" @click="showResult()">
        <span>Result</span>
      </a>

        <button v-if="accounts.length === 0" @click="connectToMetamask()">
            Connect to Metamask
        </button>
        <button v-else @click="disconnect()">Disconnect</button>
        <div class="small-text"> {{accounts[0]}}</div>
         <div class="small-text"> Balance: {{balance}} ETH </div>

    </Push>

    <div id="page-wrap">
        <div v-if = "showhome" id="home_wrap" class="container">
            <h1>{{home_message}}</h1>
            <div class="medium-text">Using the Kadenang Ginto - Decentralized App</div>
            <div>___</div>
            <div> STEP 1: Install Metamask extension</div>
            <div> STEP 2: Select Ropsten Test Network at metamask</div>
            <div> STEP 3: Get RopstenETH at https://faucet.metamask.io/</div>
            <div> STEP 4: Click "Connect to Metamask" button at Sidebar</div>
            <div> STEP 5: Click Vote text at Sidebar</div>
            <div> STEP 6: Click "Vote" button to your preferred candidate (You can only vote once)</div>
            <div> STEP 7: Accept gas payment (RopstenETH) at metamask</div>
            <div> STEP 8: You may check result</div>
            <div> STEP 9: Then click "Disconnect" button</div>
        </div>
        <div  v-if = "showvote" id="vote_wrap" class="container">
            <h1>{{vote_message}}</h1>
            <div>Number of candidates {{candidatesLength}}</div>
            <div v-if="errorMessage.length">{{errorMessage}}</div>
            <div class="candidate-card" v-for="(candidate, index) in candidates" :key="index">
                <div> Candidate {{index+1}}</div>
                <img :src="candidate.image" height="150"/>
                <div>Name:{{candidate.name}}</div>
                <button @click="addVote(index)">Vote</button>
            </div>
        </div>
        <div  v-if = "showresult" id="result_wrap" class="container">
            <h1>{{result_message}}</h1>
            <div>Number of candidates {{candidatesLength}}</div>
            <div v-if="errorMessage.length">{{errorMessage}}</div>
            <div class="candidate-card" v-for="(candidate, index) in candidates" :key="index">
                <div> Candidate {{index+1}}</div>
                <img :src="candidate.image" height="150"/>
                <div>Name:{{candidate.name}}</div>
                <div>Votes:{{candidate.voteCount}}</div>
            </div>
        </div>
    </div>

</div>
</template>
<script>


import {ethers, utils} from 'ethers'
import contractAbi from '../contract-abis/ballot.json'
import { Push } from 'vue-burger-menu'


export default{
    components:{
        Push
    },
    data () {
        return{
            errorMessage: '',
            candidates:[],
            candidatesLength: 0,
            contract: null,
            balance: 0,
            provider: null,
            accounts: [],
            candidate: {name: 'Pacman', voteCount: 10},
            home_message: 'Search for the ISCP President',
            vote_message: 'Vote Wisely Please',
            result_message: 'Congrats to the winner #ProudAspin',
            showhome: true,
            showvote: false,
            showresult: false
    
        }
    },
    methods:{
        showHome(){
                this.showhome = true
                this.showvote = false
                this.showresult = false
        },
        showVote(){
                this.showhome = false
                this.showvote = true
                this.showresult = false
        },
        showResult(){
                this.showhome = false
                this.showvote = false
                this.showresult = true
        },
        disconnect(){
            this.accounts = []
            this.balance = 0
            this.provider = null
            this.contract = null
            this.candidates = []
            this.candidatesLength = 0;
            this.errorMessage = "";
        },
        async getCandidates(){
            this.candidatesLength = await this.contract.getCandidatesLength()
            this.candidates = []
            for (var i = 0; i < this.candidatesLength; i++) {
                var candidate = await this.contract.candidates(i)
                var _candidate = {
                    name: candidate.name,
                    voteCount: candidate.voteCount,
                    image: 'https://avatars.dicebear.com/api/croodles/' + candidate.name + '.svg'
                }
                this.candidates.push(_candidate)
            }
        },
        createContractInstance(){
            this.contract = new ethers.Contract(
                '0x7B2ED477E948702BE6Cba3200438fEb0454d7280',contractAbi)
            this.contract = this.contract.connect(this.provider)

            this.getCandidates()
        },
        async getBalance(){
            var rawBalance = await this.provider.getBalance(this.accounts[0])
            this.balance = utils.formatEther (rawBalance)
       },
        async connectToMetamask(){
            this.provider = new ethers.providers.Web3Provider(window.ethereum)
            this.accounts = await this.provider.send('eth_requestAccounts',[])
            this.getBalance()
            this.createContractInstance()
        },
        async addVote(candidateIndex){
           if (confirm("Are you really sure?") == true) {
                var signer = this.provider.getSigner()
                var contractWithSigner = this.contract.connect(signer)
                try{
                    var transaction = await contractWithSigner.vote(candidateIndex)
                    await transaction.wait()
                    this.getCandidates()
                    this.errorMessage = ''
                }catch(error){
                    //alert(error.data.message);
                    //this.errorMessage = error.code
                    this.errorMessage = "Sorry, voting is unavailable! Check instruction page for your safety."
                }
            }
        },
        voteCountPlusOne(){
            return this.candidate.voteCount + 1
        }
    }
}

</script>
<style scoped>
button {
    color:white;
    background: green;
    padding: 10px 20px;
    margin: 5px;
    border: none;
    border-radius:5px;
}
button.active {
    background-color: darkgreen;
}
.small-text {
    font-size:10px;
}

.medium-text {
    font-size:20px;
}

.candidate-card{
    margin:10px;
    padding: 5px;
   /* border:1px solid black;*/
    background-color: #eee;
    border-radius: 10px;
    /*box-shadow: #9a9a9a 2px 2px 3px;*/
    width: 360px;
    float:left;
}


</style>

<style>
body {
 background-color: #ddd;
}
.containerss {
  background-color: #eee;
  width: 560px;
  margin: auto;
  padding: 10px;
  border-radius: 10px;
  box-shadow: #9a9a9a 2px 2px 3px;
}



.bm-burger-button {
      position: fixed;
      width: 36px;
      height: 30px;
      left: 36px;
      top: 36px;
      cursor: pointer;
    }
    .bm-burger-bars {
      background-color: #373a47;
    }
    .line-style {
      position: absolute;
      height: 20%;
      left: 0;
      right: 0;
    }
    .cross-style {
      position: absolute;
      top: 12px;
      right: 2px;
      cursor: pointer;
    }
    .bm-cross {
      background: #bdc3c7;
    }
    .bm-cross-button {
      height: 24px;
      width: 24px;
    }
    .bm-menu {
      height: 100%; /* 100% Full-height */
      width: 0; /* 0 width - change this with JavaScript */
      position: fixed; /* Stay in place */
      z-index: 1000; /* Stay on top */
      top: 0;
      left: 0;
      background-color: rgb(63, 63, 65); /* Black*/
      overflow-x: hidden; /* Disable horizontal scroll */
      padding-top: 60px; /* Place content 60px from the top */
      transition: 0.5s; /*0.5 second transition effect to slide in the sidenav*/
    }
 
    .bm-overlay {
      background: rgba(0, 0, 0, 0.3);
    }
    .bm-item-list {
      color: #b8b7ad;
      margin-left: 10%;
      font-size: 20px;
    }
    .bm-item-list > * {
      display: flex;
      text-decoration: none;
      padding: 0.7em;
    }
    .bm-item-list > * > span {
      margin-left: 10px;
      font-weight: 700;
      color: white;
    }
</style>

