<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <span>Javascript实现的斐波那契函数耗费： {{jsFibonacci}} ms</span>
    <span>C实现的斐波那契函数耗费： {{cFibonacci}} ms</span>
  </div>
</template>

<script>
import fetch from 'node-fetch';
import wasmC from './add.c';

export default {
  name: 'App',
  data() {
    return {
      cFibonacci: '',
      jsFibonacci: '',
    };
  },
  methods: {
    async doSomething() {
      const fibonacciUrl = './f.wasm';
      const { fibonacci } = await this.getExportFunction(fibonacciUrl);
      this.cFibonacci = this.getDuring(fibonacci),
      this.jsFibonacci = this.getDuring(this.fibonacci);
    },
    getDuring(func) {
      const start = Date.now();
      func(40);
      return Date.now() - start;
    },
    fibonacci(n) {
      if (n <= 1) {
        return n;
      }
      return this.fibonacci(n - 1) + this.fibonacci(n - 2);
    },
    async getExportFunction(url) {
      const env = {
        memoryBase: 0,
        tableBase: 0,
        memory: new WebAssembly.Memory({
          initial: 256,
        }),
        table: new WebAssembly.Table({
          initial: 2,
          element: 'anyfunc',
        }),
      };
      const instance = await fetch(url).then((response) => response.arrayBuffer()).then((bytes) => WebAssembly.instantiate(bytes, { env })).then((instance) => instance.instance.exports);
      console.log(instance,'ss')
      return instance;
    },
  },
  mounted() {
    wasmC({
      global: {},
      env: {
        memoryBase: 0,
        tableBase: 0,
        memory: new WebAssembly.Memory({ initial: 256 }),
        table: new WebAssembly.Table({ initial: 0, element: 'anyfunc' }),
      },
    }).then((result) => {
    const { exports } = result.instance;
    const add = exports.add;
    const fibonacci = exports.fibonacci;
    console.log('C return value was', add(2, 3));
    console.log('Fibonacci', fibonacci(2));
    });
    this.doSomething();
  },
  components: {
  },
};
</script>

 <style>

 </style>
