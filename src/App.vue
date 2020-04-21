<template>
	<v-app>
		<v-navigation-drawer
			:mini-variant=true
			:clipped=true
			v-model="drawer"
			left
			floating
			app
			mobile-break-point="0"
			color="#121212"
		>
			<v-list three-line>

				<!-- <v-list-item class="d-item">
					<v-list-item-avatar size="28">
						<v-img :src="require('@/assets/logo.png')" ></v-img>
					</v-list-item-avatar>
				</v-list-item> -->

				<!-- <v-divider class="mx-3"></v-divider> -->

				<v-tooltip right transition="slide-x-reverse-transition" >
					<template v-slot:activator="{ on }">
						<v-list-item slot="activator" v-on="on" class="d-item" height="88" @click="nothing()">
							<v-icon>mdi-cog</v-icon>
						</v-list-item>
					</template>
					<span>settings</span>
				</v-tooltip>

				<v-tooltip right  transition="slide-x-reverse-transition">
					<template v-slot:activator="{ on }">
						<v-list-item slot="activator" v-on="on" class="d-item" height="88"
							@click="confirmDialog = true">
							<v-icon>mdi-refresh</v-icon>
						</v-list-item>
					</template>
					<span>refresh</span>
				</v-tooltip>

				<v-tooltip right  transition="slide-x-reverse-transition">
					<template v-slot:activator="{ on }">
						<v-list-item slot="activator" v-on="on"  class="d-item" height="88"
						@click="is_fullscreen ? close_fullscreen() : open_fullscreen()">
							<v-icon>mdi-fullscreen</v-icon>
						</v-list-item>
					</template>
					<span>fullscreen</span>
				</v-tooltip>

				<!-- <v-divider class="mx-3"></v-divider> -->

				<v-tooltip v-if="$vuetify.breakpoint.smAndUp" right
					transition="slide-x-reverse-transition">
					<template v-slot:activator="{ on }" v-on="on">
						<v-list-item slot="activator" class="d-item" height="88"
						@click="infoDialog = true">
							<v-icon>mdi-information</v-icon>
						</v-list-item>
					</template>
					<span>toggle fullscreen</span>
				</v-tooltip>
			</v-list>
		</v-navigation-drawer>

		<div class="t-container" :style="{ 'margin-top': `${-correct_text.length / 4 }px` }">
			<div class="t-text" ref="t-text">
				<span class="t-success">{{ correct_text + interim_text }}</span>
				<span class="t-fail">{{ incorrect_text }}</span>
				<!-- eslint-disable-next-line max-len -->
				<span>{{ leftover_text.slice((correct_text + interim_text).length + incorrect_text.length) }}</span>
			</div>
			<v-text-field
				solo
				class="t-input"
				hide-details
				height="80"
				v-model="user_input"
				:maxlength="incorrect_text.length > 20 ? user_input.length : '50'"
			></v-text-field>
			<div class="t-bottom-gradient"></div>
			<div class="t-wpm">
				<div class="t-chart-container">
					<span class="t-abs-wpm">{{ wpm }}</span>
					<Chart
						:options="chart_options"
						ref="highcharts1"
					></Chart>
				</div>
			</div>
		</div>

		<v-dialog
			v-model="dialog" :overlay="false"
			max-width="500px"
			transition="dialog-transition"
		>
			<v-card class="d-card" color="#3d322d">
				<v-card-title>
					Are you sure you want to reload the scene?
				</v-card-title>
				<v-card-text>
					All objects will be removed from the scene... forever.
				</v-card-text>
				<v-divider></v-divider>
				<v-card-actions>
					<v-spacer></v-spacer>
					<v-btn text @click="confirmDialog = false">cancel</v-btn>
					<v-btn text @click="removeGeometry()">confirm</v-btn>
				</v-card-actions>
			</v-card>
		</v-dialog>
	</v-app>
</template>

<script>
import Highcharts from 'highcharts';
import { Chart } from 'highcharts-vue';
// import exporting from 'highcharts/modules/exporting';
// import offline_exporting from 'highcharts/modules/offline-exporting';
import stockInit from 'highcharts/modules/stock';
import { Draggable } from 'draggable-vue-directive';
import material from './assets/material';
// import simplebar from 'simplebar-vue';
// import 'simplebar/dist/simplebar.min.css';

// exporting(Highcharts);
// offline_exporting(Highcharts);
stockInit(Highcharts);

export default {
	name: 'App',
	directives: {
		Draggable,
	},
	components: {
		Chart,
	},
	data: () => ({
		polling: null,
		is_fullscreen: false,
		is_init: false,
		dialog: false,
		drawer: true,
		user_input: '',
		correct_text: '',
		interim_text: '',
		incorrect_text: '',
		leftover_text: '',
		active_string: '',
		word_array: [],
		current_failure: [],
		active_word: 0,
		wpm: 0,
		time: 0,
		failures: 0,
		chart_options: {
			chart: {
				type: 'spline',
				styledMode: true,
				events: {
					load() {
						//
					},
				},
			},
			time: {
				useUTC: false,
			},
			title: {
				text: '',
			},
			xAxis: {
				type: 'datetime',
				tickPixelInterval: 150,
				visible: false,
			},
			yAxis: {
				min: 0,
				max: 250,
				visible: false,
			},
			legend: {
				enabled: false,
			},
			plotOptions: {
				spline: {
					dataLabels: {
						enabled: true,
					},
				},
			},
			credits: {
				enabled: false,
			},
			series: [
				{
					type: 'spline',
					name: 'Random data',
					fillOpacity: 1,
					data: [[(new Date()).getTime(), 0]],
				},
				{
					type: 'pie',
					name: 'Total consumption',
					data: [
						{
							name: 'Miss',
							y: 0,
							color: '#f45b5b',
						},
						{
							name: 'Correct',
							y: 0,
							color: '#90ee7e',
						},
					],
					center: [40, 40],
					size: 100,
					showInLegend: false,
					dataLabels: {
						enabled: false,
					},
				},
			],
		},
	}),
	created() {
		this.$vuetify.theme.dark = true;
		this.leftover_text = material[1].text;
	},
	mounted() {
		this.word_array = this.leftover_text.split(' ');
	},
	methods: {
		init() {
			const { chart } = this.$refs.highcharts1;
			this.polling = setInterval(() => {
				this.time += 0.5;
				this.wpm = Math.round((this.correct_text.length / 5) / (this.time / 60));
				if (this.time % 2 === 0) {
					chart.series[0].addPoint([(new Date()).getTime(), this.wpm], true, this.time > 60);
					chart.series[1].setData([this.failures, this.correct_text.length]);
				}
			}, 500);
		},
		async open_fullscreen() {
			const elem = document.documentElement;

			if (elem.requestFullscreen) {
				elem.requestFullscreen();
			} else if (elem.mozRequestFullScreen) {
				elem.mozRequestFullScreen();
			} else if (elem.webkitRequestFullscreen) {
				elem.webkitRequestFullscreen();
			} else if (elem.msRequestFullscreen) {
				elem.msRequestFullscreen();
			}
			this.is_fullscreen = !this.is_fullscreen;
		},
		async close_fullscreen() {
			if (document.exitFullscreen) {
				document.exitFullscreen();
			} else if (document.mozCancelFullScreen) {
				document.mozCancelFullScreen();
			} else if (document.webkitExitFullscreen) {
				document.webkitExitFullscreen();
			} else if (document.msExitFullscreen) {
				document.msExitFullscreen();
			}
			this.is_fullscreen = !this.is_fullscreen;
		},
		// This function divides the text into lines as they appear on screen
		async get_all_strings(text) {
			const canvas = document.createElement('canvas');
			const context = canvas.getContext('2d');
			context.font = '22px Roboto';

			// eslint apparently is braindead
			// eslint-disable-next-line no-unused-vars
			let width = 0;
			let current_string = '';
			const all_words = [];

			text.split(' ').forEach((x, i, a) => {
				const sentence = current_string.length ? `${current_string} ${x}` : x;
				const text_width = context.measureText(sentence).width + (0.35 * (all_words.length - 2));

				if (i === a.length - 1) {
					if (text_width > 885) {
						all_words.push(current_string);
						all_words.push(x);
					} else {
						all_words.push(sentence);
					}
				} else if (text_width > 885) {
					all_words.push(current_string);
					current_string = x;
					width = context.measureText(x).width;
				} else {
					current_string = sentence;
					width = text_width;
				}
			});
			return all_words;
		},
	},
	watch: {
		user_input(value) {
			if (!this.is_init) {
				this.init();
				this.is_init = true;
			}
			let v = value.includes(' ') ? `${value.split(' ')[0]} ` : value;
			if (v === `${this.word_array[this.active_word]} `) {
				this.correct_text += v;
				if (this.active_word === this.word_array.length - 1) {
					// end of material
					// eslint-disable-next-line no-alert
					alert('You won!');
				} else {
					// prepare new word
					this.active_word += 1;
					this.incorrect_text = '';
					this.user_input = this.user_input.slice(v.length);
				}
			} else {
				v = value;
				// eslint-disable-next-line no-unused-vars
				let correct_text = '';
				for (let i = 0; i < v.length; i++) {
					if (v[i] === this.word_array[this.active_word][i]) {
						correct_text += v[i];
					} else {
						const miss = JSON.stringify([v[i], i]);
						if (this.current_failure !== miss) {
							this.current_failure = miss;
							this.failures++;
						}
						break;
					}
				}
				this.interim_text = correct_text;
				this.incorrect_text = this.leftover_text.slice(
					(this.correct_text + correct_text).length,
					(this.correct_text + correct_text).length + (v.length - correct_text.length),
				);
			}
		},
	},
	beforeDestroy() {
		clearInterval(this.polling);
	},
};
</script>

<style lang="scss">
@import './assets/chart-themes/dark-unica.scss';
@import './assets/global.scss';
</style>
