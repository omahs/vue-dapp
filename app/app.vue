<script lang="ts" setup>
import { BrowserWalletConnector, useVueDapp, type ConnWallet } from '@vue-dapp/core'
import { VueDappModal } from '@vue-dapp/modal'
import '@vue-dapp/modal/dist/style.css'
import { WalletConnectConnector } from '@vue-dapp/walletconnect'
import { CoinbaseWalletConnector } from '@vue-dapp/coinbase'
import { INFURA_ID } from './constants'

import { darkTheme, lightTheme, type GlobalThemeOverrides } from 'naive-ui'
import { useAppStore } from './stores/appStore'

const lightThemeOverrides: GlobalThemeOverrides = {
	common: {
		primaryColor: '#42b883',
		successColor: darkTheme.common.successColor,
	},
}

useHead({
	titleTemplate: title => {
		if (title) return `${title} - Vue Dapp`
		else return 'Vue Dapp - Essential Web3 Tools for Vue Developers'
	},
})

const { addConnectors, watchWalletChanged, watchDisconnect, onDisconnect, onAccountsChanged } = useVueDapp()
if (process.client) {
	addConnectors([
		new BrowserWalletConnector(),
		new WalletConnectConnector({
			projectId: 'd1e65611568666138126d315c0bafd7d',
			chains: [1],
			showQrModal: true,
			qrModalOptions: {
				themeMode: 'light',
				themeVariables: undefined,
				desktopWallets: undefined,
				walletImages: undefined,
				mobileWallets: undefined,
				enableExplorer: true,
				privacyPolicyUrl: undefined,
				termsOfServiceUrl: undefined,
			},
		}),
		new CoinbaseWalletConnector({
			appName: 'Vue Dapp',
			jsonRpcUrl: `https://mainnet.infura.io/v3/${INFURA_ID}`,
		}),
	])
}
if (process.client) {
	onAccountsChanged(() => {
		console.log('Disconnected')
	})
}

const { setWallet, resetWallet } = useEthers()

watchWalletChanged(
	async (wallet: ConnWallet) => {
		setWallet(wallet.provider)
	},
	{
		immediate: true,
	},
)

watchDisconnect(() => {
	resetWallet()
})

const hideConnectingModal = computed(() => {
	const route = useRoute()
	if (route.path === '/eip-6963') return true
	return false
})

const { darkMode } = storeToRefs(useAppStore())
</script>

<template>
	<n-config-provider :theme="lightTheme" :theme-overrides="lightThemeOverrides">
		<NuxtLayout>
			<NuxtLoadingIndicator />

			<NuxtPage />

			<VueDappModal
				:dark="darkMode"
				auto-connect
				auto-connect-browser-wallet-if-solo
				:hideConnectingModal="hideConnectingModal"
			/>
		</NuxtLayout>
	</n-config-provider>
</template>
