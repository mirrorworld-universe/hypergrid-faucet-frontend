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

      <div class="important">
        To maintain adequate balances for all users, the Faucet distributes 1 Test hSOL every 8 hours.
      </div>

      <vue-turnstile ref="turnstile" site-key="0x4AAAAAAAzMCQ0jOq3M6PGr" v-model="token" />

      <div class="confirm">
        <a-button type="primary" size="large" block :loading="loading" :disabled="disabled" @click="handleClaim">
          Confirm Airdrop
        </a-button>
      </div>
    </div>
  </section>
</template>

<script lang="ts" setup>
import { onMounted, watchEffect, ref, h } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { message, notification } from 'ant-design-vue';
import apis from '@/apis';
import utils from '@/utils';
import VueTurnstile from 'vue-turnstile';

const route = useRoute();
const router = useRouter();

const amount = '1';
const addressVal = ref('');
const token = ref('');
const loading = ref(false);
const disabled = ref(false);
const turnstile: any = ref(null);

const networkList = ref([
  {
    label: 'Testnet',
    value: 'testnet.v1',
    rpcApi: 'https://api.testnet.v1.sonic.game',
    faucetApi: 'https://faucet-api-hssn-v1.sonic.game',
    explorer: (tx) => `https://explorer-hssn.testnet.sonic.game/txs/${tx}`
  }
]);
const networkVal: any = ref(networkList.value[0].value);

watchEffect(() => {
  if (route.query.network) {
    const network = networkList.value.find((item: any) => item.value === route.query.network);
    if (network) {
      networkVal.value = route.query.network;
      disabled.value = false;
      if (turnstile.value) turnstile.value.reset();
    }
  }
  if (route.query.wallet) {
    addressVal.value = route.query.wallet as string;
  }
});

const handleChange = (value: string) => {
  if (loading.value) return;
  router.push({ query: { network: value } });
};

const handleClaim = async () => {
  if (loading.value || !addressVal.value || !token.value) return;

  loading.value = true;
  const network = networkList.value.find((item: any) => item.value === networkVal.value);
  const url = `${network?.faucetApi}/airdrop/${addressVal.value}/${amount}/${token.value}`;
  apis
    .getAirdrop(url)
    .then((res: any) => {
      console.log('getAirdrop', res.data);
      loading.value = false;
      if (res.data.data) {
        if (res.data.err) return message.error(res.data.err);
        const txhashMatch = res.data.data.match(/txhash:\s*(\w+)/);
        const tx = txhashMatch ? txhashMatch[1] : null;
        disabled.value = true;
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
          res.data.err ==
          'Usage:\n  hypergrid-ssnd tx bank send [from_key_or_address] [to_address] [amount] [flags]\n\nFlags:\n  -a, --account-number uint      The account number of the signing account (offline mode only)\n      --aux                      Generate aux signer data instead of sending a tx\n  -b, --broadcast-mode string    Transaction broadcasting mode (sync|async) (default "sync")\n      --chain-id string          The network chain ID\n      --dry-run                  ignore the --gas flag and perform a simulation of a transaction, but don\'t broadcast it (when enabled, the local Keybase is not accessible)\n      --fee-granter string       Fee granter grants fees for the transaction\n      --fee-payer string         Fee payer pays fees for the transaction instead of deducting from the signer\n      --fees string              Fees to pay along with transaction; eg: 10uatom\n      --from string              Name or address of private key with which to sign\n      --gas string               gas limit to set per-transaction; set to "auto" to calculate sufficient gas automatically. Note: "auto" option doesn\'t always report accurate results. Set a valid coin value to adjust the result. Can be used instead of "fees". (default 200000)\n      --gas-adjustment float     adjustment factor to be multiplied against the estimate returned by the tx simulation; if the gas limit is set manually this flag is ignored  (default 1)\n      --gas-prices string        Gas prices in decimal format to determine the transaction fee (e.g. 0.1uatom)\n      --generate-only            Build an unsigned transaction and write it to STDOUT (when enabled, the local Keybase only accessed when providing a key name)\n  -h, --help                     help for send\n      --keyring-backend string   Select keyring\'s backend (os|file|kwallet|pass|test|memory) (default "os")\n      --keyring-dir string       The client Keyring directory; if omitted, the default \'home\' directory will be used\n      --ledger                   Use a connected Ledger device\n      --node string              <host>:<port> to CometBFT rpc interface for this chain (default "tcp://localhost:26657")\n      --note string              Note to add a description to the transaction (previously --memo)\n      --offline                  Offline mode (does not allow any online functionality)\n  -o, --output string            Output format (text|json) (default "json")\n  -s, --sequence uint            The sequence number of the signing account (offline mode only)\n      --sign-mode string         Choose sign mode (direct|amino-json|direct-aux|textual), this is an advanced feature\n      --timeout-height uint      Set a block timeout height to prevent the tx from being committed past a certain height\n      --tip string               Tip is the amount that is going to be transferred to the fee payer on the target chain. This flag is only valid when used with --aux, and is ignored if the target chain didn\'t enable the TipDecorator\n  -y, --yes                      Skip tx broadcasting prompt confirmation\n\nGlobal Flags:\n      --home string         directory for config and data (default "/home/ubuntu/.hypergrid-ssn")\n      --log_format string   The logging format (json|plain) (default "plain")\n      --log_level string    The logging level (trace|debug|info|warn|error|fatal|panic|disabled or \'*:<level>,<key>:<level>\') (default "info")\n      --log_no_color        Disable colored logs\n      --trace               print out full stack trace on errors\n\ndecoding bech32 failed: invalid bech32 string length 4\n'
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
        disabled.value = true;
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
      span {
        font-family: Orbitron;
        font-weight: 200;
        font-size: 16px;
        margin-right: 20px;
      }
    }
  }
}

@media screen and (max-width: 750px) {
  .home {
    .box {
      width: 90vw;
    }
  }
}
</style>
