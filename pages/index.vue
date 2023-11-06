<template>
	<div>
		<h1>Blockchain Nodes operated by @visvirial</h1>
		<div class="text-center">
			Last update: {{ ((now - lastUpdate) / 1000).toFixed(1) }} secs ago.
		</div>
		<v-table>
			<thead>
				<tr>
					<th>Chain</th>
					<th>Endpoint</th>
					<th>Local Height</th>
					<th>Remote Height</th>
					<th>Difference</th>
					<th>Remote Source</th>
				</tr>
			</thead>
			<tbody>
				<tr v-for="(chain, name) in chains">
					<td>{{ chain.name }}</td>
					<td>{{ chain.endpoint }}</td>
					<td><BlockHeight :value=chain.local /></td>
					<td><BlockHeight :value=chain.remote /></td>
					<td><HeightDifference :local=chain.local :remote=chain.remote :tolerance=chain.tolerance /></td>
					<td><a :href=chain.source.url target="_blank">{{ chain.source.name }}</a></td>
				</tr>
			</tbody>
		</v-table>
		<hr />
		<footer>
			<div>Copyright &copy; <a href="https://twitter.com/visvirial" target="_blank">@visvirial</a> all rights reserved.</div>
		</footer>
	</div>
</template>

<script lang=ts>

import { defineComponent } from 'vue';
import { ethers } from 'ethers';

const Chain = {
	name: String,
	endpoint: String,
	local: Number,
	remote: Number,
	tolerance: Number,
	source: {
		name: String,
		url: String,
	},
};

export default defineComponent({
	data() {
		return {
			now: Date.now(),
			lastUpdate: 0,
			chains: {
				btc: {
					name: 'Bitcoin (Mainnet)',
					endpoint: 'https://btc.node.visvirial.com/',
					local: -1,
					remote: -1,
					tolerance: 5,
					source: {
						name: 'mempool',
						url: 'https://mempool.space/',
					},
				},
				tbtc: {
					name: 'Bitcoin (Testnet)',
					endpoint: 'https://tbtc.node.visvirial.com/',
					local: -1,
					remote: -1,
					tolerance: 5,
					source: {
						name: 'mempool',
						url: 'https://mempool.space/testnet',
					},
				},
				sbtc: {
					name: 'Bitcoin (Signet)',
					endpoint: 'https://sbtc.node.visvirial.com/',
					local: -1,
					remote: -1,
					tolerance: 5,
					source: {
						name: 'mempool',
						url: 'https://mempool.space/signet',
					},
				},
				mona: {
					name: 'Monacoin (Mainnet)',
					endpoint: 'https://mona.node.visvirial.com/',
					local: -1,
					remote: -1,
					tolerance: 10,
					source: {
						name: 'Trezor Monacoin Explorer',
						url: 'https://blockbook.electrum-mona.org/',
					},
				},
				tmona: {
					name: 'Monacoin (Testnet)',
					endpoint: 'https://tmona.node.visvirial.com/',
					local: -1,
					remote: -1,
					tolerance: 10,
					source: {
						name: 'Trezor Monacoin Testnet Explorer',
						url: 'https://testnet-blockbook.electrum-mona.org/',
					},
				},
				eth: {
					name: 'Ethereum',
					endpoint: 'https://eth.node.visvirial.com/',
					local: -1,
					remote: -1,
					tolerance: 30,
					source: {
						name: 'Ankr',
						url: 'https://ankr.com/',
					},
				},
				bsc: {
					name: 'Binance Smart Chain',
					endpoint: 'https://bsc.node.visvirial.com/',
					local: -1,
					remote: -1,
					tolerance: 50,
					source: {
						name: 'Ankr',
						url: 'https://ankr.com/',
					},
				},
				tron: {
					name: 'Tron',
					endpoint: 'https://tron.node.visvirial.com/',
					local: -1,
					remote: -1,
					tolerance: 100,
					source: {
						name: 'TRONSCAN',
						url: 'https://tronscan.org/',
					},
				},
			},
		};
	},
	methods: {
		updateLastUpdate() {
			this.now = Date.now();
		},
		async updateAll() {
			await Promise.all([
				// Bitcoin (Mainnnet).
				(async () => {
					this.chains.btc.local = (await (await fetch('https://btc.node.visvirial.com/rest/chaininfo.json')).json()).blocks;
					this.chains.btc.remote = await (await fetch('https://mempool.space/api/blocks/tip/height')).json();
				})(),
				// Bitcoin (Testnet).
				(async () => {
					this.chains.tbtc.local = (await (await fetch('https://tbtc.node.visvirial.com/rest/chaininfo.json')).json()).blocks;
					this.chains.tbtc.remote = await (await fetch('https://mempool.space/testnet/api/blocks/tip/height')).json();
				})(),
				// Bitcoin (Signet).
				(async () => {
					this.chains.sbtc.local = (await (await fetch('https://sbtc.node.visvirial.com/rest/chaininfo.json')).json()).blocks;
					this.chains.sbtc.remote = await (await fetch('https://mempool.space/signet/api/blocks/tip/height')).json();
				})(),
				// Monacoin (Mainnet).
				(async () => {
					this.chains.mona.local = (await (await fetch('https://mona.node.visvirial.com/rest/chaininfo.json')).json()).blocks;
					this.chains.mona.remote = (await (await fetch('https://blockbook.electrum-mona.org/api/')).json()).backend.blocks;
				})(),
				// Monacoin (Testnet).
				(async () => {
					this.chains.tmona.local = (await (await fetch('https://tmona.node.visvirial.com/rest/chaininfo.json')).json()).blocks;
					this.chains.tmona.remote = (await (await fetch('https://testnet-blockbook.electrum-mona.org/api/')).json()).backend.blocks;
				})(),
				// Ethereum.
				(async () => {
					const providerLocal = new ethers.JsonRpcProvider('https://eth.node.visvirial.com/');
					this.chains.eth.local = await providerLocal.getBlockNumber();
					const providerRemote = new ethers.JsonRpcProvider('https://rpc.ankr.com/eth');
					this.chains.eth.remote = await providerRemote.getBlockNumber();
				})(),
				// BSC.
				(async () => {
					const providerLocal = new ethers.JsonRpcProvider('https://bsc.node.visvirial.com/');
					this.chains.bsc.local = await providerLocal.getBlockNumber();
					const providerRemote = new ethers.JsonRpcProvider('https://rpc.ankr.com//bsc');
					this.chains.bsc.remote = await providerRemote.getBlockNumber();
				})(),
				// Tron.
				(async () => {
					this.chains.tron.local = (await (await fetch('https://tron.node.visvirial.com/wallet/getnowblock')).json()).block_header.raw_data.number;
					this.chains.tron.remote = (await (await fetch('https://apilist.tronscan.org/api/block/statistic')).json()).whole_block_count;
				})(),
				/*
				(async () => {
				})(),
				*/
			]);
			this.lastUpdate = Date.now();
		},
	},
	async mounted() {
		this.updateAll();
		setInterval(this.updateAll, 10_000);
		setInterval(this.updateLastUpdate, 100);
	},
});

</script>

