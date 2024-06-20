<template>
  <section class="home">
    <div class="box">
      <div class="title">
        <div>Request Airdrop</div>
      </div>

      <div class="title2">
        <a-button type="primary" @click="utils.handleCopy(rpc)"><CheckCircleOutlined />{{ rpc }}</a-button>
      </div>
      <div class="inputbox">
        <a-input v-model:value="addressVal" placeholder="Wallet Address" size="large" />
      </div>
      <div class="text">
        <span>Amount: </span>
        <div class="tag">{{ amount }}</div>
      </div>
      <vue-turnstile site-key="0x4AAAAAAAc5PDWj7n4NdQwS" v-model="token" />
      <div class="confirm">
        <a-button type="primary" size="large" block :loading="loading" @click="handleClaim">Confirm Airdrop</a-button>
      </div>
    </div>
  </section>
</template>

<script lang="ts" setup>
import { onMounted, onBeforeUnmount, computed, ref, h } from 'vue';
import { message, notification } from 'ant-design-vue';
import { CheckCircleOutlined } from '@ant-design/icons-vue';
import apis from '@/apis';
import utils from '@/utils';
import VueTurnstile from 'vue-turnstile';

const addressVal = ref('');
const loading = ref(false);

let lastClaimTime = ref(0);
const amount = '10';
const token = ref(
  '0.Qu2Aqn7BdnAfFOaSy9v_1a7e9zQ2uh7KUaxblpyMOQZcQyXGyKsXI23ha2z9MFOavKdEWRnIzdpf5izabmSi3KR8gM65_V-AS_GfPQjjqPnw1AzfxItEk20jgG5725GalYwShVKRDQ5-L2PBCxMKxMcGlO-_WufpYyyE66kJPI7mCpjhJWPRAQpOsJgjelEyv8b8600QG3diNmVn0zg9u1p3nHbM5T56AVJsxIxBjxx_WcATJVrNkiGgBNFKrNmtcQWLK3nFUp-KBON74dBS4RIT1GUuztNjvFsU2rw2ToSLh90VehoURWQY9TjTsdtaOOjGE5BGwZ8yYczNIXHJqZ2ttuhIlNG5yfg28l_NZRTzFogCqhNpDhb7jFG-xaPw0V9AA6MlZMTK_2owTicv22_jFv8vBrp1E0fXRV0wdgngfa6bv0Vxc7Pmmz2tbti8.2WnkrOivdigua5ttFQT74A.7a06cc29ff2e8aad52fabde3268111b30af781f4e2bde1781fe1d4e378fca740'
);

const rpc = 'https://rpc.hypergrid.dev';
const explorer = 'https://explorer.hypergrid.dev/tx/';

onMounted(() => {
  lastClaimTime.value = Number(localStorage.getItem('lastClaimTime')) || 0;
});

const handleClaim = () => {
  if (loading.value) return;
  if (!addressVal.value) return;
  if (!token.value) return;
  // console.log('addressVal', addressVal.value);

  const currentTime = Date.now();
  const timeDiff = currentTime - lastClaimTime.value;

  if (timeDiff >= 5 * 60 * 1000) {
    lastClaimTime.value = currentTime;
    loading.value = true;
    apis
      .getAirdrop(addressVal.value, amount, token.value)
      .then((res: any) => {
        console.log('getAirdrop', res.data);
        loading.value = false;

        if (res.data.data) {
          if (res.data.data.error) return message.error(res.data.data.error);
          localStorage.setItem('lastClaimTime', lastClaimTime.value.toString());
          const tx = res.data.data.replace(/\n/g, '').replace('Signature: ', '');
          console.log('tx', tx);
          notification.success({
            message: 'Airdrop was successful!',
            description: () => {
              return h('a', {
                href: explorer + tx,
                target: '_blank',
                innerHTML: explorer + utils.formatAddr(tx)
              });
            },
            duration: null
          });
        } else {
          if (
            res.data.err == "error: Invalid value for '<RECIPIENT_ADDRESS>': No such file or directory (os error 2)\n"
          ) {
            message.error('Invalid address');
          } else {
            message.error(res.data.err);
          }
        }
      })
      .catch((error) => {
        loading.value = false;
        console.log(error);
        message.error('Airdrop failed');
      });
  } else {
    message.info('Please wait five minutes to receive it again!');
  }
};
</script>

<style lang="scss" scoped>
.home {
  .box {
    width: 500px;
    height: auto;
    margin: 0 auto;
    border: 1px solid #ffffff1a;
    border-radius: 10px;
    padding: 20px;
    position: relative;
    > div {
      margin-bottom: 30px;
      &:last-child {
        margin-bottom: 0;
      }
    }
    .title {
      font-family: Orbitron;
      font-size: 20px;
      div:nth-child(2) {
        font-size: 12px;
      }
    }
    .title2 {
      font-family: Orbitron;
      font-size: 12px;
      position: absolute;
      right: 20px;
      top: 20px;
    }
    .text {
      font-family: Orbitron;
      font-weight: 200;
      font-size: 16px;
      display: flex;
      align-items: center;
      .tag {
        width: 50px;
        height: 40px;
        line-height: 40px;
        border-radius: 5px;
        text-align: center;
        font-family: Manrope;
        font-weight: 700;
        font-size: 18px;
        border: 1px solid #0000ff;
        box-shadow: 0 0 3px 3px rgba(0, 0, 255, 0.3);
      }
      span {
        margin-right: 20px;
      }
    }
  }
}
.ant-btn-default {
  color: #fff;
}

@media screen and (max-width: 750px) {
  .home {
    .box {
      width: 90vw;
    }
  }
}
</style>
