<template>
  <div class="home container mx-auto">
    <div v-if="!this.account">
      <span>Error: Account not been set!</span>
    </div>
    <div v-if="!this.client">

    </div>
    <div v-if="this.account">
      <h1 class="py-2">
        Test contracts
        <span v-if="!this.client" class="text-sm text-red">(connecting to client...)</span>
        <span v-if="this.client" class="text-sm text-green">(client connected to {{this.host}})</span>
      </h1>
      <div class="mt-8 -mx-2">
        <div class="w-full p-4 bg-grey-light rounded-sm shadow">
          <h2 class="py-2">
            Sophia Contract's Code:
          </h2>

          <div class="relative">
            <codemirror v-model="contractCode" :options="cmOption"></codemirror>
            <div class='absolute pin-b pin-r'>
              <select v-model="cmOption.keyMap" >
                <option :key="idx" v-for='(m, idx) in keymaps'>
                  {{m}}
                </option>
              </select>
            </div>
            </div>

          <div v-if="compileError">
            <label class="text-xs block mb-1 text-red">Errors</label>
            <textarea v-model="compileError" class="h-64 w-full border border-solid border-black font-mono bg-black text-red"></textarea>
          </div>

          <button v-if="this.client" class="mt-2 rounded-full bg-black hover:bg-purple text-white p-2 px-4" @click="onCompile">Compile</button>
        </div>

        <div class="flex mt-8 mb-8" v-if="byteCode">
          <div class="w-1/2 p-4 bg-grey-light rounded-sm shadow">
            <h2 class="py-2">
              Byte Code
              <div class="w-full text-xs" v-bind:class="{ 'text-red' : !deployedDataObj, 'text-green' : deployedDataObj }">
                {{deployInfo}}
              </div>
            </h2>
            <textarea v-model="byteCode" class="h-16 w-full border border-solid border-black font-mono bg-black text-white text-xs"></textarea>

            <div class="mt-2 mb-2" v-if="deployError">
              <label class="text-xs block mb-1 text-red">Deploy Errors:</label>
              <div class="w-full border border-solid border-black font-mono bg-black text-red text-sm">
                {{deployError}}
              </div>
            </div>

            <div class="flex -mx-2 mt-4 mb-4">
              <div class="mx-2">
                <label class="text-xs block mb-1" for="deployFunc">Function</label>
                <input v-model="deployFunc" class="w-full p-2" id="deployFunc" type="text" value="init" disabled>
              </div>
              <div class="mx-2">
                <label class="text-xs block mb-1" for="deployArgs">Arguments</label>
                <input v-model="deployArgs" class="w-full p-2" id="deployArgs" type="text" placeholder="arguments">
              </div>
            </div>
            <div class="flex -mx-2 mt-4 mb-4">
              <div class="mx-2 w-1/5">
                <label class="text-xs block mb-1" for="dDeposit">Deposit</label>
                <input v-model="deployOpts.deposit" class="w-full p-2" id="dDeposit" type="text" placeholder="deposit">
              </div>
              <div class="mx-2 w-1/5">
                <label class="text-xs block mb-1" for="dGasPrice">Gas Price</label>
                <input v-model="deployOpts.gasPrice" class="w-full p-2" id="dGasPrice" type="text" placeholder="gas price">
              </div>
              <div class="mx-2 w-1/5">
                <label class="text-xs block mb-1" for="dAmount">Amount</label>
                <input v-model="deployOpts.amount" class="w-full p-2" id="dAmout" type="text" placeholder="amount">
              </div>
              <div class="mx-2 w-1/5">
                <label class="text-xs block mb-1" for="dFee">Fee</label>
                <input v-model="deployOpts.fee" class="w-full p-2" id="dFee" type="text" placeholder="fee">
              </div>
              <div class="mx-2 w-1/5">
                <label class="text-xs block mb-1" for="dGas">Gas</label>
                <input v-model="deployOpts.gas" class="w-full p-2" id="dGas" type="text" placeholder="gas">
              </div>

              <input v-model="deployOpts.callData" class="w-full p-2" type="hidden" value="callData">

            </div>

            <button class="py-2 rounded-full bg-black hover:bg-purple text-white p-2 px-4" @click="onDeploy">Deploy</button>
          </div>

          <div class="w-1/2 p-4 bg-grey-light rounded-sm shadow">
            <h2 class="py-2">
              ⬅ Call Static Function
            </h2>
            <div class="flex -mx-2 mt-4 mb-4">
              <div class="mx-2 w-1/2">
                <label class="text-xs block mb-1" for="staticFunc">Function</label>
                <input v-model="staticFunc" class="w-full p-2" id="staticFunc" type="text" placeholder="function">
              </div>
              <div class="mx-2 w-1/2">
                <label class="text-xs block mb-1" for="staticArgs">Arguments</label>
                <input v-model="staticArgs" class="w-full p-2" id="staticArgs" type="text" placeholder="arguments">
              </div>
            </div>

            <div class="mt-2 mb-2" v-if="callStaticRes && !callStaticError">
              <label class="text-xs block mb-1">Call Result</label>
              <div class="w-full text-white bg-black text-xs mb-4 p-4 overflow-x-scroll font-mono">
                {{callStaticRes}}
              </div>
            </div>
            <div class="mt-2 mb-2" v-if="callStaticError">
              <label class="text-xs block mb-1 text-red">Errors</label>
              <textarea v-model="callStaticError" class="h-48 w-full text-red bg-black text-xs mb-4 p-4 overflow-x-scroll font-mono"></textarea>
            </div>

            <button class="py-2 rounded-full bg-black hover:bg-purple text-white p-2 px-4" @click="onCallStatic">Call Static</button>
          </div>
        </div>

        <div v-if="deployedDataObj && byteCode" class="w-full p-4 bg-grey-light rounded-sm shadow mb-8">
          <h2 class="py-2">
            ⬆ Call Function
          </h2>
          <div class="flex -mx-2 mt-4 mb-4">
            <div class="mx-2 w-1/5">
              <label class="text-xs block mb-1" for="cDeposit">Deposit</label>
              <input v-model="callOpts.deposit" class="w-full p-2" id="cDeposit" type="text" placeholder="deposit">
            </div>
            <div class="mx-2 w-1/5">
              <label class="text-xs block mb-1" for="cGasPrice">Gas Price</label>
              <input v-model="callOpts.gasPrice" class="w-full p-2" id="cGasPrice" type="text" placeholder="gas price">
            </div>
            <div class="mx-2 w-1/5">
              <label class="text-xs block mb-1" for="cAmount">Amount</label>
              <input v-model="callOpts.amount" class="w-full p-2" id="cAmout" type="text" placeholder="amount">
            </div>
            <div class="mx-2 w-1/5">
              <label class="text-xs block mb-1" for="cFee">Fee</label>
              <input v-model="callOpts.fee" class="w-full p-2" id="cFee" type="text" placeholder="fee">
            </div>
            <div class="mx-2 w-1/5">
              <label class="text-xs block mb-1" for="cGas">Gas</label>
              <input v-model="callOpts.gas" class="w-full p-2" id="cGas" type="text" placeholder="gas">
            </div>

            <input v-model="callOpts.callData" class="mx-2 w-1/2 p-2" type="hidden" value="callData">

          </div>
          <div class="flex -mx-2 mt-4 mb-4">
            <div class="mx-2 w-1/2">
              <label class="text-xs block mb-1" for="func">Function</label>
              <input v-model="nonStaticFunc" class="w-full p-2" id="func" type="text" placeholder="function">
            </div>
            <div class="mx-2 w-1/2">
              <label class="text-xs block mb-1" for="args">Arguments</label>
              <input v-model="nonStaticArgs" class="w-full p-2" id="args" type="text" placeholder="arguments">
            </div>
          </div>

          <div class="mt-2 mb-2" v-if="callRes && !callError">
            <label class="text-xs block mb-1">Call Result</label>
            <div class="w-full text-white bg-black text-xs mb-4 p-4 overflow-x-scroll font-mono">
              {{callRes}}
            </div>
          </div>
          <div class="mt-2 mb-2" v-if="callError">
            <label class="text-xs block mb-1 text-red">Errors</label>
            <textarea v-model="callError" class="h-16 w-full border border-solid border-black font-mono bg-black text-white mb-2"></textarea>
          </div>

          <button class="py-2 rounded-full bg-black hover:bg-purple text-white p-2 px-4" @click="onCallDataAndFunction">Call Function</button>
          <span v-if="waitingCall" class="text-sm text-red">Calling Function...</span>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
import Ae, { Contract, Wallet } from '@aeternity/aepp-sdk'
import account from '../account.js'

import { codemirror } from 'vue-codemirror'

export default {
  name: 'Home',
  components: {
    codemirror
  },
  data () {
    return {
      keymaps: [
        'sublime',
        'vim',
        'emacs'
      ],
      cmOption: {
        keyMap: 'sublime',
        tabSize: 4,
        styleActiveLine: true,
        lineNumbers: true,
        mode: 'text/javascript',
        theme: 'base16-dark'
      },
      contractCode: `contract Identity =
  type state = ()
  function main(x : int) = x`,
      account: account,
      byteCode: '',
      client: false,
      host: 'https://sdk-testnet.aepps.com',
      deployedDataObj: false,
      deployInfo: '',
      minedData: false,
      miningStatus: '',
      wallet: false,
      byteCodeObj: {},
      deployFunc: 'init',
      deployArgs: '()',
      staticFunc: 'main',
      staticArgs: '(1)',
      nonStaticFunc: '',
      nonStaticArgs: '',
      deployOpts: {
        deposit: 1,
        gasPrice: 1,
        amount: 1,
        fee: 1,
        gas: 40000000,
        callData: ''
      },
      callOpts: {
        deposit: 1,
        gasPrice: 1,
        amount: 1,
        fee: 1,
        gas: 40000000,
        callData: ''
      },
      callRes: '',
      callError: '',
      deployError: '',
      compileError: '',
      callStaticRes: '',
      callStaticError: '',
      waitingCall: false
    }
  },
  props: {
    query: {
      type: Object
    }
  },
  watch: {
    keyMap (mapName) {
      try {
        window.localStorage.setItem('cmkeyMap', mapName)
      } catch (e) {
        /* handle error */
      }
    }
  },
  computed: {
    keyMap () {
      return this.cmOption.keyMap
    }
  },
  methods: {
    async compile (client, code) {
      console.log(`Compiling contract...`)
      const wallet = Wallet.create(this.client, account)
      const contract = Contract.create(this.client, { wallet })
      try {
        console.log(`Compiled!`)
        return await contract.compile(code)
      } catch (err) {
        this.compileError = err
        console.log(err)
      }
    },
    async deploy (options = {}) {
      console.log(`Deploying contract...`, account)
      try {
        return this.byteCodeObj.deploy(options)
      } catch (err) {
        console.log(err)
      }
    },
    async callStatic (func, args = '1') {
      console.log(`calling static func ${func} with args ${args}`)
      try {
        return this.byteCodeObj.call(
          func,
          { args: args }
        )
      } catch (err) {
        console.log(err)
      }
    },
    async callContract (func, args, opts) {
      console.log(`calling a function on a deployed contract with func: ${func}, args: ${args} and options:`, opts)

      try {
        return this.deployedDataObj.call(
          func,
          {
            args,
            opts
          }
        )
      } catch (err) {
        console.log(err)
      }
    },
    onCompile () {
      this.compileError = ''
      this.callError = ''
      this.deployError = ''
      this.callStaticError = ''
      this.deployedData = false
      this.deployInfo = ''
      this.minedData = false
      this.miningStatus = false
      this.byteCode = false

      this.compile(this.client, this.contractCode)
        .then(byteCodeObj => {
          this.byteCodeObj = byteCodeObj
          this.byteCode = byteCodeObj.bytecode
        })
    },
    onDeploy () {
      this.deployInfo = 'Deploying and checking for mining status...'
      this.miningStatus = true
      const extraOpts = {
        'owner': account.pub,
        'code': this.contractCode,
        'vmVersion': 1,
        'nonce': 0,
        'ttl': 9999999,
        'options': {
          'initState' : this.deployArgs
        }
      }
      const opts = Object.assign(extraOpts, this.deployOpts)

      this.deploy(opts) // this waits until the TX is mined
        .then(data => {
          this.deployInfo = `Deployed, and mined (Address: ${data.address})`
          this.miningStatus = false
          this.deployedDataObj = data
        })
        .catch(err => {
          this.deployError = `${err}`
        })
    },
    onCallStatic () {
      if (this.staticFunc && this.staticArgs) {
        this.callStatic(this.staticFunc, this.staticArgs)
          .then(data => {
            this.callStaticRes = data
            this.callStaticError = ''
          })
          .catch(err => {
            this.callStaticError = `${err}`
          })
      } else {
        this.callStaticError = 'Please enter a Function and 1 or more Arguments.'
      }
    },
    onCallDataAndFunction () {
      const extraOpts = {
        'owner': account.pub,
        'code': this.contractCode,
        'vmVersion': 1,
        'nonce': 0,
        'ttl': 9999999
      }
      const opts = Object.assign(extraOpts, this.callOpts)

      if (this.nonStaticFunc && this.nonStaticArgs) {
        this.waitingCall = true
        this.callContract(this.nonStaticFunc, this.nonStaticArgs, opts)
          .then(data => {
            this.callRes = data
            this.callError = ''
            this.waitingCall = false
          })
          .catch(err => {
            this.callError = `${err}`
            this.waitingCall = false
          })
      } else {
        this.callError = 'Please enter a Function and 1 or more Arguments.'
      }
    }
  },
  async mounted () {
    try {
      const mapName = window.localStorage.getItem('cmkeyMap')
      if (mapName) this.cmOption.keyMap = mapName
    } catch (e) {
      /* handle error */
    }
    this.client = await Ae.create(this.host, {debug: true})
  }
}
</script>

<style scoped lang="css">
</style>
