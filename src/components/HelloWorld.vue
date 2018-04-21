<template>
  <div>
    <h1>{{ msg }}</h1>
    <h1>{{ productName }}</h1>
    <h1>{{ manufacturerName }}</h1>
    <el-button type="text" @click="click">USB start</el-button>
    <el-button type="text" @click="cap">Cap</el-button>
  </div>
</template>

<script>
let num = 0
let device = null
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      productName: '',
      manufacturerName: ''
    }
  },
  methods: {
    cap: async function (event) {
      if (device === null) {
        return
      }
      console.log(device.configuration)
      // USBEndpoint {endpointNumber: 1, direction: "in", type: "bulk", packetSize: 512}
      // USBEndpoint {endpointNumber: 1, direction: "out", type: "bulk", packetSize: 512}
      // USBEndpoint {endpointNumber: 2, direction: "in", type: "interrupt", packetSize: 28}
      if (device.configuration === null) {
        console.log('selectConfiguration')
        await device.selectConfiguration(0)
      }
      function int2byte (data) {
        const array = []
        for (let i = 0; i < 4; i++) {
          array.push(data & 0xFF)
          data = data >>> 8
        }
        return array
      }
      console.log('interfaces:', device.configuration.interfaces)
      await device.claimInterface(0)
      await device.transferOut(1, new Uint8Array([
        0x10, 0, 0, 0,
        0x01, 0x00,
        0x02, 0x10,
        ...int2byte(num),
        1, 0, 0, 0]
      ))
      num++
      await device.transferOut(1, new Uint8Array([
        0x0c, 0, 0, 0,
        0x01, 0x00,
        0x01, 0x10,
        ...int2byte(num)]
      ))
      num++
      await device.transferOut(1, new Uint8Array([
        0x14, 0, 0, 0,
        0x01, 0x00,
        0x0e, 0x10, // PTP_OC_InitiateCapture
        ...int2byte(num),
        0, 0, 0, 0,
        0, 0, 0, 0]
      ))
      num++
      await device.transferOut(1, new Uint8Array([
        0x0c, 0, 0, 0,
        0x01, 0x00,
        0x03, 0x10, // PTP_OC_CloseSession
        ...int2byte(num)]
      ))
      num++
    },
    click: async function (event) {
      try {
        device = await navigator.usb.requestDevice({ filters: [{ vendorId: 0x05ca }] })
        console.log(device.productName)
        console.log(device.manufacturerName)
        this.productName = device.productName
        this.manufacturerName = device.manufacturerName
        await device.open()
        /*
        const senddata = [

        ]
        //await device.transferOut(1, );
        */
      } catch (err) {
        console.error(err)
      }
    }
  },
  mounted () {
    console.log('mounted')
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
