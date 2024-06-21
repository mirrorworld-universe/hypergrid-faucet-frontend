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

const rpc = 'https://rpc.hypergrid.dev';
const explorer = 'https://explorer.hypergrid.dev/tx/';
const amount = '1';
const addressVal = ref('');
const token = ref('');
const loading = ref(false);

const handleClaim = () => {
  if (loading.value) return;
  if (!addressVal.value) return;

  const currentTime = Date.now();
  const lastClaimTime = Number(localStorage.getItem('lastClaimTime')) || 0;
  const timeDiff = currentTime - lastClaimTime;

  if (timeDiff >= 86400000) {
    loading.value = true;
    apis
      .getAirdrop(addressVal.value, amount, token.value)
      .then((res: any) => {
        console.log('getAirdrop', res.data);
        loading.value = false;

        if (res.data.data) {
          if (res.data.data.error) return message.error(res.data.data.error);
          localStorage.setItem('lastClaimTime', currentTime.toString());
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
        console.error(error);
        if (error.status == 429) {
          message.error(error.data.message);
        } else {
          message.error('Airdrop failed');
        }
      });
  } else {
    message.info('Please allow 24 hours to receive it again!');
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
