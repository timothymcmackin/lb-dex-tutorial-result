<script lang="ts">
  import { onMount } from "svelte";
  import { BeaconWallet } from "@taquito/beacon-wallet";
  import store, { type TezosAccountAddress } from "../store";
  import { rpcUrl, network } from "../config";
  import { shortenHash, fetchBalances } from "../utils";

  let connectedNetwork = "";
  let walletIcon = "";
  let walletName = "";

  const connectWallet = async () => {
    if (!$store.wallet) {
      const wallet = new BeaconWallet({
        name: "Tezos dev portal app tutorial",
        preferredNetwork: network,
      });
      store.updateWallet(wallet);
    }

    await $store.wallet.requestPermissions({
      network: { type: network, rpcUrl },
    });
    const userAddress = (await $store.wallet.getPKH()) as TezosAccountAddress;
    store.updateUserAddress(userAddress);
    $store.Tezos.setWalletProvider($store.wallet);
    // Get account info
    await getWalletInfo($store.wallet);
    // Fetch the user's XTZ, tzBTC and SIRS balances
    const res = await fetchBalances($store.Tezos, userAddress);
    if (res) {
      store.updateUserBalance("XTZ", res.xtzBalance);
      store.updateUserBalance("tzBTC", res.tzbtcBalance);
      store.updateUserBalance("SIRS", res.sirsBalance);
    } else {
      store.updateUserBalance("XTZ", null);
      store.updateUserBalance("tzBTC", null);
      store.updateUserBalance("SIRS", null);
    }
  };

  onMount(async () => {
    const wallet = new BeaconWallet({
      name: "Tezos dev portal app tutorial",
      preferredNetwork: network,
    });
    store.updateWallet(wallet);
    const activeAccount = await wallet.client.getActiveAccount();
    if (activeAccount) {
      const userAddress = (await wallet.getPKH()) as TezosAccountAddress;
      store.updateUserAddress(userAddress);
      $store.Tezos.setWalletProvider(wallet);
      await getWalletInfo(wallet);
      // fetches user's XTZ, tzBTC and SIRS balances
      const res = await fetchBalances($store.Tezos, userAddress);
      if (res) {
        store.updateUserBalance("XTZ", res.xtzBalance);
        store.updateUserBalance("tzBTC", res.tzbtcBalance);
        store.updateUserBalance("SIRS", res.sirsBalance);
      } else {
        store.updateUserBalance("XTZ", null);
        store.updateUserBalance("tzBTC", null);
        store.updateUserBalance("SIRS", null);
      }
    }
  });

  const disconnectWallet = async () => {
    $store.wallet.client.clearActiveAccount();
    store.updateWallet(undefined);
    store.updateUserAddress(undefined);
    connectedNetwork = "";
    walletIcon = "";
  };
</script>
