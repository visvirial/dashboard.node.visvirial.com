<template>
	<div>
		<h1>Blockchain Nodes operated by @visvirial</h1>
		<div class="text-center">
			Last update: {{ lastUpdate < 0 ? 'unknown' : ((now - lastUpdate) / 1000).toFixed(1) + ' secs ago' }}.
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
			lastUpdate: -1,
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
				arb: {
					name: 'Arbitrum',
					endpoint: 'https://arb.node.visvirial.com/',
					local: -1,
					remote: -1,
					tolerance: 50,
					source: {
						name: 'Ankr',
						url: 'https://ankr.com/',
					},
				},
				op: {
					name: 'Optimism',
					endpoint: 'https://op.node.visvirial.com/',
					local: -1,
					remote: -1,
					tolerance: 50,
					source: {
						name: 'Ankr',
						url: 'https://ankr.com/',
					},
				},
				avax: {
					name: 'Avalanche',
					endpoint: 'https://avax.node.visvirial.com/',
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
			const promises: Promise<void>[] = [];
			// Bitcoin.
			const mempoolPrefix: { [chain: string]: string } = { btc: '', tbtc: 'testnet/', sbtc: 'signet/' };
			for(const chain in mempoolPrefix) {
				promises.push((async () => {
					try {
						this.chains[chain].local = (await (await fetch(`https://${chain}.node.visvirial.com/rest/chaininfo.json`)).json()).blocks;
					} catch (e) {
						this.chains[chain].local = -1;
					}
					try {
						this.chains[chain].remote = await (await fetch(`https://mempool.space/${mempoolPrefix[chain]}api/blocks/tip/height`)).json();
					} catch (e) {
						this.chains[chain].remote = -1;
					}
				})());
			}
			// Monacoin.
			const electrumMonaPrefix: { [chain: string]: string } = { mona: '', tmona: 'testnet-' };
			for(const chain in electrumMonaPrefix) {
				promises.push((async () => {
					try {
						this.chains[chain].local = (await (await fetch(`https://${chain}.node.visvirial.com/rest/chaininfo.json`)).json()).blocks;
					} catch (e) {
						this.chains[chain].local = -1;
					}
					try {
						this.chains[chain].remote = (await (await fetch(`https://${electrumMonaPrefix[chain]}blockbook.electrum-mona.org/api/`)).json()).backend.blocks;
					} catch (e) {
						this.chains[chain].remote = -1;
					}
				})());
			}
			// EVM.
			const ankrPostfix: { [chain: string]: string } = { eth: 'eth', bsc: 'bsc', arb: 'arbitrum', op: 'optimism', avax: 'avalanche' };
			for(const chain in ankrPostfix) {
				promises.push((async () => {
					try {
						const providerLocal = new ethers.JsonRpcProvider(`https://${chain}.node.visvirial.com/` + (chain === 'avax' ? 'ext/bc/C/rpc' : ''));
						this.chains[chain].local = await providerLocal.getBlockNumber();
					} catch (e) {
						this.chains[chain].local = -1;
					}
					try {
						const providerRemote = new ethers.JsonRpcProvider(`https://rpc.ankr.com/${ankrPostfix[chain]}`);
						this.chains[chain].remote = await providerRemote.getBlockNumber();
					} catch (e) {
						this.chains[chain].remote = -1;
					}
				})());
			}
			// Tron.
			promises.push((async () => {
				try {
					this.chains.tron.local = (await (await fetch('https://tron.node.visvirial.com/wallet/getnowblock')).json()).block_header.raw_data.number;
				} catch (e) {
					this.chains.tron.local = -1;
				}
				try {
					this.chains.tron.remote = (await (await fetch('https://apilist.tronscan.org/api/block/statistic')).json()).whole_block_count;
				} catch (e) {
					this.chains.tron.remote = -1;
				}
			})());
			await Promise.all(promises);
			this.lastUpdate = Date.now();
		},
	},
	async mounted() {
		await this.updateAll();
		setInterval(this.updateAll, 60_000);
		setInterval(this.updateLastUpdate, 100);
	},
});

</script>

