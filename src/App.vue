<script setup>
import { ref, computed } from 'vue'
import OpenAI from 'openai'

const system = ref("")
const message = ref("")
const prompt_tokens = ref(0)
const completion_tokens = ref(0)
const input_tokens = ref(0)
const output_tokens = ref(0)

const name = ref("")
const tel = ref("")
const requirements = ref("")

const messages = ref([])
const systems = ref([
  { role: 'system', content: 'あなたは株式会社アガガの電話受付担当の田中です。'},
  { role: 'system', content: '電話対応だとして応対してください。'},
  { role: 'system', content: '応答は80文字以内で行ってください。'},
  { role: 'system', content: '今日の日付は' + new Date().toLocaleString('ja-JP') + 'です。' },
  { role: 'system', content: '自社の社員の名前を呼ぶ時には敬称は省いてください。'},
  { role: 'system', content: '自社の社員は田中、鈴木、佐々木の3名です。'},
  { role: 'system', content: '電話を繋いだり、対応を確認したりできないことはしないでください。'},
  { role: 'system', content: '名前と要件と連絡先だけ聞き出して折り返し電話する旨を伝えて会話を終了してください。'},
])
const functions = ref([
{
      name: "regist_name",
      description: "名前が確認できた場合に呼び出し。",
      parameters: {
        type: "object",
        properties: {
          name: {
            type: "string",
            description: "名前, e.g. 佐々木",
          },
        },
        required: [ "name" ],
      }
    },
    {
      name: "regist_tel",
      description: "電話番号が確認できた場合に呼び出し。",
      parameters: {
        type: "object",
        properties: {
          tel: {
            type: "string",
            description: "電話番号, e.g. 07011112222",
          },
        },
        required: [ "tel" ],
      }
    },
    {
      name: "regist_requirements",
      description: "要件が確認できた場合に呼び出し。",
      parameters: {
        type: "object",
        properties: {
          requirements: {
            type: "string",
            description: "要件",
          },
        },
        required: [ "requirements" ],
      }
    },
])

async function send(forced = false) {
  if (!message.value && !forced)
    return
  // OpenAI初期化
  const openai = new OpenAI({
    organization: 'org-xxxxxxxxxxxxxxxxxxxxxxxx',
    apiKey: 'sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx',
    dangerouslyAllowBrowser: true   // ブラウザ上で強制実行するためのフラグ
  })
  // 要求
  if (message.value) {
    messages.value.push({ role: 'user', content: message.value })
    message.value = ""
  }
  var prompt = systems.value.concat(ref([{ role: 'system', content: system.value }]).value.concat(messages.value))
  const completion = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: prompt,
    temperature: 1.0,
    functions: functions.value,
    function_call: 'auto'
  })
  // 応答
  prompt_tokens.value = Number(completion.usage.prompt_tokens)
  completion_tokens.value = Number(completion.usage.completion_tokens)
  input_tokens.value += prompt_tokens.value
  output_tokens.value += completion_tokens.value
  completion.choices.forEach(choice => {
    if (choice.message.content !== null) {
      // テキスト応答
      messages.value.push(choice.message)
    } else if (choice.finish_reason === 'function_call') {
      // ファンクションコールバック
      var isRegist = false
      switch(choice.message.function_call.name) {
        case 'regist_name':         // 名前
          const { name } = JSON.parse(choice.message.function_call.arguments)
          isRegist = regist_name(name)
          break;
        case 'regist_tel':          // 電話番号
          const { tel } = JSON.parse(choice.message.function_call.arguments)
          isRegist = regist_tel(tel)
          break;
        case 'regist_requirements': // 要件
          const { requirements } = JSON.parse(choice.message.function_call.arguments)
          isRegist = regist_requirements(requirements)
          break;
      }
      if (isRegist) {
        // ファンクションコールバックの場合はもう一度呼び出さないとテキスト応答を返してくれない。
        send(true)
      }
    }
  })
}

const input_amount = computed(() => {
  return (input_tokens.value / 1000.0) * 0.03
})
const output_amount = computed(() => {
  return (output_tokens.value / 1000.0) * 0.06
})


// 名前登録
function regist_name(val) {
  name.value = val
  messages.value.push({ role: 'function', name: 'regist_name', content: val })
  return true
}

// 電話番号登録
function regist_tel(val) {
  tel.value = val
  messages.value.push({ role: 'function', name: 'regist_tel', content: val })
  return true
}

// 要件登録
function regist_requirements(val) {
  requirements.value = val
  messages.value.push({ role: 'function', name: 'regist_requirements', content: val })
  return true
}

</script>

<template>
  <div class="header">
    <span class="prompt">SYSTEM PROMPT</span><textarea v-model="system" rows="3" cols="80" placeholder="AIの役割を指示してください"></textarea>
  </div>
  <span style="color: red">名前 : </span>{{ name }}, <span style="color: red">電話番号 : </span>{{ tel }}, <span style="color: red">要件 : </span>{{ requirements }} 
  <div v-for="(item) in messages">
    {{ item.role }}: {{ item.content }}
  </div>
  <div class="footer">
    <div>
      <span class="prompt">USER MESSAGE</span><textarea v-model="message" rows="3" cols="80"></textarea>
      <button @click="send">送信</button>
    </div>
    <div>
      prompt_tokens: {{ prompt_tokens }}, completion_tokens: {{  completion_tokens }}<br/>
      input_tokens: {{ input_tokens }} (${{ input_amount.toFixed(4) }}), output_tokens: {{ output_tokens }} (${{ output_amount.toFixed(4) }})<br/>
      total: ${{ (input_amount + output_amount).toFixed(4) }}
    </div>
  </div>
</template>

<style scoped>
.prompt {
  display: inline-block;
  width: 130px;
  vertical-align: top;
}

button {
  margin-left: 5px;
  vertical-align: top;
}

.header {
  width: 100%;
  border-bottom: 1px solid;
}

.footer {
  width: 100%;
  padding-top: 5px;
  position: absolute;
  bottom: 0;
}
</style>
