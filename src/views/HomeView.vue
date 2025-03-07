<template>
  <section class="home">
    <div class="box">
      <div class="title">
        <div>Request Airdrop</div>
      </div>

      <div class="title2">
        <a-select
          ref="select"
          class="select-option"
          style="width: 220px"
          v-model:value="networkVal"
          :options="networkList"
          @change="handleChange"></a-select>
      </div>
      <div class="inputbox">
        <a-input v-model:value="addressVal" placeholder="Wallet Address" size="large" />
      </div>
      <div class="text">
        <span>Amount: </span>
        <div class="tag">{{ amount }}</div>
      </div>

      <template v-if="networkVal == 'testnet.v1'">
        <div class="text">
          <span>Mainnet SOL balance: </span>
          <div class="balance">
            {{ utils.formatNumber(solBalance, 2) }}
            <template v-if="addressVal">
              <CloseCircleFilled v-if="solBalance < 0.01" />
              <CheckCircleFilled v-else />
            </template>
          </div>
        </div>
        <div class="important">You need at least 0.01 SOL in your wallet on Solana Mainnet to access the faucet.</div>
      </template>

      <div class="important">
        To maintain adequate balances for all users, the Faucet distributes 0.5 Test SOL every 8 hours.
      </div>

      <vue-turnstile ref="turnstile" site-key="0x4AAAAAAAc6HG1RMG_8EHSC" v-model="token" />

      <div class="confirm">
        <a-button
          type="primary"
          size="large"
          block
          :loading="loading"
          :disabled="!addressVal || !token || (networkVal == 'testnet.v1' && solBalance < 0.01)"
          @click="handleClaim">
          Confirm Airdrop
        </a-button>
      </div>
    </div>
  </section>
</template>

<script lang="ts" setup>
import { onMounted, watchEffect, ref, h, watch } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { message, notification } from 'ant-design-vue';
import { CheckCircleFilled, CloseCircleFilled } from '@ant-design/icons-vue';
import apis from '@/apis';
import utils from '@/utils';
import VueTurnstile from 'vue-turnstile';
import { Connection, PublicKey } from '@solana/web3.js';

const route = useRoute();
const router = useRouter();

const amount = '5';
const addressVal = ref('');
const token = ref('');
const loading = ref(false);
const turnstile: any = ref(null);
const solBalance = ref(0);

const networkList = ref([
  // {
  //   label: 'Devnet',
  //   value: 'devnet',
  //   rpcApi: 'https://api.devnet.sonic.game',
  //   faucetApi: 'https://faucet-api.sonic.game',
  //   explorer: (tx) => `https://explorer.sonic.game/tx/${tx}`
  // },
  // {
  //   label: 'Testnet V0',
  //   value: 'testnet.v0',
  //   rpcApi: 'https://api.testnet.v0.sonic.game',
  //   faucetApi: 'https://faucet-api-grid-1.sonic.game',
  //   explorer: (tx) => `https://explorer.sonic.game/tx/${tx}?cluster=testnet.v0`
  // },
  {
    label: 'Testnet V1',
    value: 'testnet.v1',
    rpcApi: 'https://api.testnet.v1.sonic.game',
    faucetApi: 'https://faucet-api-grid-1-v1.sonic.game',
    explorer: (tx) => `https://explorer.sonic.game/tx/${tx}?cluster=testnet.v1`
  }
]);
const networkVal: any = ref(networkList.value[0].value);

watchEffect(() => {
  if (route.query.network) {
    const network = networkList.value.find((item: any) => item.value === route.query.network);
    if (network) {
      networkVal.value = route.query.network;
      if (turnstile.value) turnstile.value.reset();
    }
  }
  if (route.query.wallet) {
    addressVal.value = route.query.wallet as string;
  }
});

watch(
  [addressVal, networkVal],
  ([_addressVal, _networkVal]) => {
    getBalance();
  },
  { immediate: true }
);

async function getBalance() {
  console.log('addressVal', addressVal.value);
  console.log('networkVal', networkVal.value);

  if (!addressVal.value) return (solBalance.value = 0);
  if (networkVal.value !== 'testnet.v1') return;
  const connection = new Connection('https://solana-mainnet.g.alchemy.com/v2/6BzorHtAxXZTGVDOxuzQ9rCk_q_qpXgj');
  try {
    const publicKey = new PublicKey(addressVal.value);
    const balance = await connection.getBalance(publicKey);
    solBalance.value = balance / 1e9;
    console.log('solBalance', solBalance.value);
  } catch (error) {
    console.log('Unable to get mainnet wallet balance');
  }
}

const handleChange = (value: string) => {
  if (loading.value) return;
  router.push({ query: { network: value } });
};

const handleClaim = async () => {
  if (
    loading.value ||
    !addressVal.value ||
    !token.value ||
    (networkVal.value == 'testnet.v1' && solBalance.value < 0.01)
  )
    return;

  loading.value = true;
  const network = networkList.value.find((item: any) => item.value === networkVal.value);
  const url = `${network?.faucetApi}/airdrop/${addressVal.value}/${amount}/${token.value}`;
  apis
    .getAirdrop(url)
    .then((res: any) => {
      console.log('getAirdrop', res.data);
      loading.value = false;

      if (res.data.data) {
        if (res.data.data.error) return message.error(res.data.data.error);
        const tx = res.data.data.replace(/\n/g, '').replace('Signature: ', '');
        console.log('tx', tx);

        notification.success({
          message: 'Airdrop was successful!',
          description: () => {
            return h('a', {
              href: network?.explorer(tx),
              target: '_blank',
              innerHTML: network?.explorer(utils.formatAddr(tx))
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
      if (error.status == 401) {
        message.error(error.data.error);
      } else if (error.status == 429) {
        message.error(error.data.message);
      } else {
        message.error('Airdrop failed');
      }
    });
};
</script>

<style lang="scss" scoped>
.home {
  .important {
    width: 100%;
    margin: 0 auto 10px;
    font-family: Orbitron;
    font-size: 16px;
    color: #fff;
  }
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
        color: #fff;
        border: 1px solid #0000ff;
        box-shadow: 0 0 3px 3px rgba(0, 0, 255, 0.3);
      }
      .balance {
        font-family: Manrope;
        font-weight: 700;
        font-size: 18px;
        color: #fff;
        display: flex;
        align-items: center;
        .anticon {
          margin-left: 20px;
          font-size: 20px;
        }
      }
      span {
        font-family: Orbitron;
        font-weight: 200;
        font-size: 16px;
        margin-right: 20px;
      }
    }
  }
}

.anticon.anticon-close-circle {
  color: #de1303;
}
.anticon.anticon-check-circle {
  color: #0aa937;
}

@media screen and (max-width: 750px) {
  .home {
    .box {
      width: 90vw;
    }
  }
}
</style>
