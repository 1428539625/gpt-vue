<script>
   	export default {
 	 	name: 'myForm',
  
data(){
    return {
        list:['已支持GPT3.5模型','可用自己的key 将无限制','欢迎大家使用GPT3.5'],
        listTop:0,
        selected  :"1"
    }
},
mounted(){
    this.setScroll();
},
methods:{
    setScroll(){
        var t = setInterval(()=>{
            this.listTop-=1;
            if(this.listTop===-40){
                this.list.push(this.list.splice(0,1)[0]);//删除数组第一个元素，插入到数组后面
                this.listTop=0;//重置top
                clearInterval(t);//取消定时器
                //设置一个2秒后执行的定时器,重新开启滚动
                setTimeout(()=>{
                    this.setScroll();
                },2000)
            }
        },20)
    },
getvaluemethod: function(event) {
					//获取当前选中的值，并重新赋值
          this.selected = event.target.value;
        }
       

}
     }
     
</script>
<script setup>
  import axios from 'axios'
  import Dialogs from './components/Dialogs.vue'

  import { nextTick, getCurrentInstance, onMounted, reactive, ref, watch } from 'vue'
  const placeholder = ref(null)

  const { proxy } = getCurrentInstance();
  const messages = reactive([])
  const dark = ref(false)
  const input = ref('')
  const apikey = ref('')
  const tokens = reactive({
    total_available: 0,
    total_granted: 0,
    total_used: 0,
  })
  const loading = ref(false)

  const restart = () => {
    location.reload()
  }

  const queryTokens = async () => {
    const result = await axios({
      url: 'https://agent-openai.ccrui.dev/dashboard/billing/credit_grants',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${apikey.value}`,
      },
    }).then(res => res.data)

    tokens.total_available = parseFloat(result.total_available).toFixed(2)
    tokens.total_used = parseFloat(result.total_used).toFixed(2)
    tokens.total_granted = parseFloat(result.total_granted).toFixed(2)
  }

  const saveKey = () => {
    if (apikey.value) {
      localStorage.setItem('api_key', apikey.value)
      queryTokens()
    } else {
      localStorage.removeItem('api_key')
      tokens.total_available = 0
      tokens.total_used = 0
      tokens.total_granted = 0
    }
  }

  onMounted(() => {
    const localKey = localStorage.getItem('api_key')
    const darkMode = localStorage.getItem('dark_mode')

    if (darkMode) {
      dark.value = darkMode === 'true'
    }

    if (localKey) {
      apikey.value = localKey
      queryTokens()
    }
    
  })
//方法：
let formData = new FormData();
function fileUpload(event) {
      console.log(event);
      let file = event.target.files;
     
      formData.append("file", file[0]);
      formData.append("model","whisper-1")
      
 console.log("modele  "+formData.get("model")+"file  "+formData.get("file"))


    }
  const sendMessage = () => {
  var myselect=document.getElementById("apps");
   var index = myselect.selectedIndex;
  let values = myselect.options[index].value;
    if (loading.value) return

    const lastCall = +localStorage.getItem('last_call')

    // 没有设置 apikey，且两次请求间隔小于设定的间隔
    if (!apikey.value && lastCall && rate_limit && Date.now() - +lastCall < rate_limit * 1000) {
      proxy.$toast.show('你的请求频率太快了，请等3s。使用自己的APIKey无频率限制。', {
        type: 'warning',
        maxToasts: 1,
        duration: 6000,
      })
      return
    }

    loading.value = true
    messages.push(
      {
        who: 'ME',
        message: input.value
        
      },
      {
        who: 'AI',
        values:values,
        message: '',
      },
    )

    nextTick(() => {
      placeholder.value.scrollIntoView({ smooth: true })
    })

let bbb=null
let ccc=null
if(values==1){
   const prompt = messages
      .filter(msg => !!msg.message)
      .map(msg => `${msg.who.toUpperCase()}: ${msg.message.trim()}`)
      .join('\n')
      .concat('\nAI: ')
var JSONS={
 model: 'gpt-3.5-turbo',
   "messages": [{
     "role": "user",
    "content": prompt}
    ],

         max_tokens: 512,
         top_p: 1,
        // best_of: 1,
         temperature: 0.7,
         frequency_penalty: 0,
         presence_penalty: 0,
        // prompt: prompt,
        stop: ['ME:', 'AI:'],
        stream:true
}
var url="https://agent-openai.ccrui.dev/v1/chat/completions"
var types="application/json"
 bbb=JSON.stringify(JSONS)
 ccc="'Content-Type':'application/json'"
}else if(values==3){
  var ins =my.sss.value;
   const prompt =ins
  var JSONS={
 "max_tokens":1000,"temperature":0.9,"frequency_penalty":0,"presence_penalty":0,"best_of":1,"logprobs":0,"stream":true,"echo":true,
        prompt: prompt
}

var url="https://agent-openai.ccrui.dev/v1/engines/code-davinci-002/completions"
var types="application/json"
bbb=JSON.stringify(JSONS)
}else if(values==2){
   var ins =my.sss.value;
   const prompt =ins
  var JSONS={
     prompt:prompt,
      "model":"image-alpha-001",
      "n":1,
      "size":"1024x1024"
       
      
}
var url="https://agent-openai.ccrui.dev/v1/images/generations"
var types="application/json"
bbb=JSON.stringify(JSONS)
}


      fetch(url, {
      method: 'POST',
      body:bbb,
      headers: {
        'Content-Type':'application/json',
        Authorization: `Bearer ${apikey.value || "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"}`
      
      },
    })
      .then(response => {
        if (response.status === 200) {
          return response.body
        }
        throw response
      })
      .then(rb => {
        const reader = rb.getReader()

        return new ReadableStream({
          start(controller) {
            // The following function handles each data chunk
            function push() {
              // "done" is a Boolean and value a "Uint8Array"
              reader.read().then(({ done, value }) => {
                // If there is no more data to read
                if (done) {
                  console.log('done', done)
                  controller.close()
                  return
                }
                // Get the data and send it to the browser via the controller
                controller.enqueue(value)
                // Check chunks by logging to the console
                const chunkStr = new TextDecoder().decode(value)

                try {
                  const packages = chunkStr.match(/data:\s/g)
                  const chunkArray = chunkStr.split('data:').filter(str => {
                    return !!str.trim() && (str.trim() !== '[DONE]')
                  })
                 var msgText=null;
                  if(values==1){
                     msgText = chunkArray.map(chunk => JSON.parse(chunk)).map(obj => obj.choices[0].delta.content).join('')
                }else if(values==3){
                       msgText = chunkArray.map(chunk => JSON.parse(chunk)).map(obj => obj.choices[0].text).join('')
                }else if(values==2){
                
                       msgText = chunkArray.map(chunk => JSON.parse(chunk)).map(obj => obj.data[0].url).join('')
                  
                }
                 
                  const lastMessage = messages[messages.length - 1]
                  lastMessage.message += msgText
                

                  placeholder.value.scrollIntoView({ smooth: true })
                } catch (e) {
                  console.error(e)
                }

                push()
              })
            }

            push()
          },
        })
      })
      .then(stream => new Response(stream, { headers: { 'Content-Type': 'text/html' } }).text())
      .then(result => {
        /* console.log(result) */
        loading.value = false
        localStorage.setItem('last_call', Date.now())
      })
      .catch(e => {
        console.log(e)
        const lastMessage = messages[messages.length - 1]
        switch (e.status) {
          case 401: {
            lastMessage.message = '错误的 API KEY'
            break
          }
          case 403: {
            lastMessage.message = '无权访问'
            break
          }
          case 404: {
            lastMessage.message = '未找到'
            break
          }
          case 429: {
            lastMessage.message = '达到请求次数限制，请歇一会 | 使用自己key将独享'
            break
          }
          default: {
            console.log(e)
            lastMessage.message = '出错了 ' + e.message
            break
          }
        }
        loading.value = false
        localStorage.setItem('last_call', Date.now())
      })

    input.value = ''
  }

  watch(dark, newVal => {
    localStorage.setItem('dark_mode', newVal)
    if (newVal) {
      document.documentElement.classList.add('dark')
    } else {
      document.documentElement.classList.remove('dark')
    }
  })
</script>


<template>

  <div>
    <div class="relative py-20 sm:py-24 bg-white dark:bg-gray-800">
      <div class="max-w-5xl mx-auto">
        <form
          class="px-4 sm:px-6 flex flex-row items-center justify-between"
          @submit.prevent="saveKey"
        >
          <input
            type="text"
            placeholder="sk-xxx"
            v-model="apikey"
            class="px-3 py-2 text-gray-700 bg-white border rounded-md dark:bg-gray-900 dark:text-gray-300 dark:border-gray-600 focus:border-blue-400 dark:focus:border-blue-300 focus:outline-none focus:ring focus:ring-blue-300 focus:ring-opacity-40 flex-grow"
            @blur="saveKey"
          />
          <button
            type="submit"
            class="mx-2 sm:mx-4 px-3 py-2 text-sm font-medium tracking-wide text-white
capitalize transition-colors duration-300 transform bg-blue-700 rounded-md hover:bg-blue-600 focus:outline-none focus:bg-blue-600 whitespace-nowrap"
          onclick="alert('设置成功')"
          >
            设置
          </button>
          <button
            class="px-3 py-2 text-sm font-medium tracking-wide text-white capitalize transition-colors duration-300 transform bg-blue-700 rounded-md hover:bg-blue-600 focus:outline-none focus:bg-blue-600 whitespace-nowrap"
            @click="queryTokens"
          >
            查询
          </button>
          
      
     
   

        </form>
   
        <div
          v-if="apikey && tokens.total_granted"
          class="text-gray-500 px-4 sm:px-6 pt-1 flex flex-row items-center text-sm dark:text-gray-400"
        >
          <div class="pl-1 sm:pl-2">
            <span>总：</span>
            <span>${{ tokens.total_granted }}</span>
          </div>
          <div class="ml-3">
            <span>已用：</span>
            <span>${{ tokens.total_used }}</span>
          </div>
          <div class="ml-3">
            <span>剩余：</span>
            <span>${{ tokens.total_available }}</span>
          </div>
        </div>
        <div v-if="!apikey"
          class="text-gray-500 px-4 sm:px-6 pt-1 flex flex-row items-center text-sm dark:text-gray-400"
        >
          <span>不需要填key也可使用</span>
          <span v-if="$window.show_ads || $window.rate_limit">，但</span>
          <span v-if="$window.rate_limit">请求频率受限</span>
          <span v-if="$window.show_ads">有广告</span>。
          
        </div>
        
<div class="box">
    <div class="list" :style="{top:listTop+'px'}">
        <div v-for="(item,index) in list" :key="index">
        {{item}}
        </div>
    </div>
</div>
        <Dialogs :messages="messages" :loading="loading" />
        <div ref="placeholder"></div>
        <div
          class="fixed bottom-0 left-0 right-0 py-3 bg-gray-100 border-t flex items-center justify-center dark:bg-gray-800 dark:border-t-gray-900"
        >
          <form
            @submit.prevent="sendMessage"
            class="flex-grow flex flex-row justify-center items-center max-w-5xl pl-2 pr-4 sm:pl-4 sm:pr-6"
            name="my"
          >
            <button
              type="button"
              class="pl-2 pr-3 py-2 text-sm font-medium tracking-wide text-gray-500 dark:text-gray-300 focus:outline-none whitespace-nowrap"
              @click.prevent="restart"
            >
              <svg
                height="800px"
                width="800px"
                version="1.1"
                id="Layer_1"
                xmlns="http://www.w3.org/2000/svg"
                xmlns:xlink="http://www.w3.org/1999/xlink"
                viewBox="0 0 482.827 482.827"
                xml:space="preserve"
                class="w-4 h-4"
              >
                <path
                  d="M241.413,0C171.855,0,106.16,30.011,60.606,81.465v-81.1H40.913v119.433h119.433v-19.692H70.671
			c41.954-50.628,104.403-80.413,170.743-80.413c122.26,0,221.721,99.462,221.721,221.721s-99.462,221.721-221.721,221.721
			S19.692,363.673,19.692,241.413c0-18.471,2.279-36.851,6.779-54.625l-19.096-4.827C2.481,201.313,0,221.317,0,241.413
			c0,133.115,108.298,241.413,241.413,241.413s241.413-108.298,241.413-241.413S374.529,0,241.413,0z"
                />
              </svg>
            </button>


            <input
           
              type="text"
              name="sss"
              class="px-4 py-2 text-gray-700 bg-white border rounded-md mr-2 sm:mr-4 dark:bg-gray-900 dark:text-gray-300 dark:border-gray-600 focus:border-blue-400 dark:focus:border-blue-300 focus:outline-none focus:ring focus:ring-blue-300 focus:ring-opacity-40 flex-grow"
              placeholder="请输入"
              :value="input"
              @input="event => (input = event.target.value)"
            />

            <button
              class="px-4 py-2 text-sm font-medium tracking-wide text-white capitalize transition-colors duration-300 transform bg-blue-700 rounded-md hover:bg-blue-600 focus:outline-none focus:bg-blue-600 whitespace-nowrap"
              type="submit"
            >
              发送
            </button>
          
            &emsp;
              <select id ="apps" v-model="selected" @change="getvaluemethod($event)" >
     		<option value="1">语言模式</option>
    	 	<option value="2" >图片模式</option>
    	 	<option value="3">代码模式</option>
          <!-- <option value="4">文件模式</option>  -->
 	 </select>
          </form>
          		



        </div>
      </div>
    </div>
  </div>
</template>

<style scoped></style>
