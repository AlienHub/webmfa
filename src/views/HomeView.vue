<template>
  <main class="main">
    
    <div class="header">
      <div class="header-title">不安全的 MFA 验证器</div>
      <ElButton class="inputkey"  @click='addKey' plain>添 加</ElButton>
      <!-- <el-button class="downloadjson"  @click='createKey' plain>创 建</el-button> -->
    </div>
    <div v-if='addT == 0' class="addkey">
      <el-input v-model="text" placeholder="账户" class="inputn"/>
      <el-input v-model="key" placeholder="密钥" class="inputn"/>
      <div class="bu">    
        <ElButton @click='cancelAddKey'>取 消</ElButton>
        <el-button type="primary" @click='confirmAddKey'>添 加</el-button>
      </div>
    </div>
    <div v-else-if="addT == 1">
      <el-input v-model="textName" placeholder="账户" class="inputn"/>
      <div class="bu">    
        <el-button @click='cancelAddKey'>取 消</el-button>
        <el-button type="primary" @click='confirmCreateKey'>创 建</el-button>
      </div>
    </div>
    <div v-else>
      <div 
        class="acard"
        @click="copynumber(item.value.key)"
        :key="item.key"
        v-for='item in items'>
        <div class="card">
          <div class="content">
            <span class="con-title">{{ item.value.value }}</span>
            <span class="con-number">{{ item.value.key }}</span>
          </div>
          <div class="time">
            <span class="d-number">{{ remainingTime }}</span>
          </div>
        </div>
        <div style="width: 40px;">
                <DeleteFilled  class="delete" @click='deletekey($event,item.key)'/>
            </div>
      </div>
    </div>
  </main>
</template>

<script setup lang="ts">
import { TOTP } from 'otpauth';
import { ref, onMounted } from 'vue';
import { ElButton, ElInput, ElMessage } from 'element-plus';
// import ElMessage from "element-plus";
// import ElButton from "element-plus";
// import ElInput from "element-plus";
import 'element-plus/dist/index.css';
import { DeleteFilled } from '@element-plus/icons-vue';
import { v4 as uuidv4 } from 'uuid';

const remainingTime = ref(0);
const text = ref('');
const textName = ref('');
const key = ref('');
const addT = ref(0);

// 定义一个数组，用来存储localStorage的key和value
interface Item {
  key: string;
  value:{
    key: string;
    value: string | null;
  };
}
const items: Item[] = [];

const addKey = () => {
  addT.value = 0;
};

const createKey = () => {
  addT.value = 1;
};

const deletekey = (event:any,key:any) => {
  event.stopPropagation();
  console.log('delete');
  console.log(key);
  localStorage.removeItem(key);
  getKeysFromLocalStorage();
  // calculateRemainingTime();
};

const confirmCreateKey = () => {
  if (textName.value == '') {
    textName.value = 'App';
  }
  const Newtotp = new TOTP();
  const secret = Newtotp.secret;
  // localStorage.setItem(secret.base32, textName.value);
  const keydata = {
    key: secret.base32,
    value: textName.value,
  }
  const lenkey = uuidv4().substr(0, 8);
  console.log(keydata);
  console.log(lenkey)
  localStorage.setItem(lenkey.toString(), JSON.stringify(keydata));
  getKeysFromLocalStorage();
  textName.value = '';
  addT.value = 2;
};

const cancelAddKey = () => {
  addT.value = 2;
  text.value = '';
  key.value = '';
  textName.value = '';
};

const confirmAddKey = () => {
  if (text.value && key.value) {
    const keydata = {
      key: key.value,
      value: text.value,
    }
    const lenkey = uuidv4().substr(0, 8);
    localStorage.setItem(lenkey.toString(), JSON.stringify(keydata));
    getKeysFromLocalStorage();
    text.value = '';
    key.value = '';
    addT.value = 2;
  }else{
    ElMessage({
      message: '请输入账户和密钥',
      duration: 1000,
      type: 'error'
    });
  }
};

const copynumber = (item:string) => {
  navigator.clipboard.writeText(item).then(function() {
    ElMessage({
      message: '复制成功',
      duration: 1000,
      type: 'success'
    });
  }, function() {
    ElMessage({
      message: '复制失败',
      duration: 1000,
      type: 'error'
    });
  });
};

onMounted(() => {
  addT.value = 2;
  getKeysFromLocalStorage();
  calculateRemainingTime();

  // 每秒钟刷新剩余时间
  setInterval(() => { 
    calculateRemainingTime();
  }, 1000);
});

// 从localStorage中获取密钥
function getKeysFromLocalStorage() {
  items.splice(0, items.length);
  for (let i = 0; i < localStorage.length; i++) {
    const key = localStorage.key(i);
    if (key == null) {
      continue;
    }
    const data = JSON.parse(localStorage.getItem(key.toString()) || '');
    const value = data.value;
    const token = generateToken(data.key);
    items.push({
      key: key,
      value: {
        key: token,
        value: value,
      },
    });
  }
}

const generateToken = (key:string) => {
  const totp = new TOTP({ secret: key });
  const token = totp.generate();
  return token;
};

// 计算剩余时间
function calculateRemainingTime() {
  const currentTimestamp = Math.floor(Date.now() / 1000);
  const timestampRemainder = currentTimestamp % 30;
  remainingTime.value = 30 - timestampRemainder;
  // console.log(remainingTime.value);
  getKeysFromLocalStorage();
}

</script>

<style scoped>
.main {
  display: flex;
  /* 一行一列 */
  flex-direction: column;
  justify-content: flex-start;
  align-items: flex-start;
  width: 333px;
  height: 100vh;
  padding: 16px 0;
}
.header {
  /* 一行两列左右分割 */
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 333px;
  height: 48px;
  margin-bottom: 16px;
  padding: 8px;
  border-radius: 4px;
  /* background-color: var(--el-color-primary-light-7); */
}

.header-title {
  /* 加粗，主题色 */
  font-weight: bold;
  color: var(--el-color-primary-dark-2);
}

.inputkey {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 32px;
  width: 124px;
  border-radius: 4px;
  background-color: #fff;
}
.inputn {
  margin-bottom: 16px;
  padding: 0 16px;
}
.downloadjson {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 32px;
  width: 124px;
  border-radius: 4px;
  background-color: #fff;
}

.addkey {
  display: flex;
  /* 一行一列 */
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 333px;
}
.bu {
  padding: 16px 0;
  display: flex;
  /* 靠右排 */
  justify-content: flex-end;
  align-items: center;
  width: 333px;
}
.acard {
  margin-bottom: 24px;
}

.acard:hover {
  cursor: pointer;
  box-shadow: 0px 2px 8px  rgba(0, 0, 0, 0.15);
}

.card {
  width: 333px;
  height: 109px;
  padding: 16px;
  border-radius: 5px;
  background: rgba(255, 255, 255, 1);
  border: 1px solid rgba(0, 0, 0, 0.15);
  /* box-shadow: 0px 2px 8px  rgba(0, 0, 0, 0.15); */
  /* flex布局，一行两列左右分割 */
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.delete {
  display: none;
  /* 靠左下角 */
  justify-content: flex-end;
  align-items: flex-end;
  margin: -32px 0 0 298px;
  width: 24px;
  color: rgba(0, 0, 0, 0.15);
}

.acard:hover .delete {
      display: flex;
}

.delete:hover {
  cursor: pointer;
  color: rgba(0, 0, 0, 0.5);
}

.content {
  display: flex;
  /* 一行展示一个数据 */
  flex-direction: column;
  justify-content: center;
  align-items: left;
}
.con-title {
  /** 文本1 */
  font-size: 20px;
  font-weight: 400;
  letter-spacing: 0px;
  line-height: 23.17px;
  color: rgba(0, 0, 0, 1);
  text-align: left;
  vertical-align: top;
}
.con-number {
  /** 文本1 */
  font-size: 32px;
  font-weight: 700;
  letter-spacing: -0.17px;
  line-height: 46.34px;
  color: var(--el-color-primary-dark-2);
  text-align: left;
  vertical-align: top;
}
.d-number {
  /** 文本1 */
  font-size: 28px;
  font-weight: 700;
  letter-spacing: 0px;
  line-height: 40.54px;
  color: var(--el-color-primary);
  text-align: right;
  vertical-align: top;
}

</style>