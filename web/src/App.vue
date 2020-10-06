<template>
	<v-app>
		<v-app-bar app color="primary" dark>
			<v-toolbar-title class="headline pointer mr-3 hidden-sm-and-down">
				<router-link
					:to="{ path: '/', query: { rootId: $route.query.rootId } }"
					tag="span"
					>{{ title }}</router-link
				>
			</v-toolbar-title>
			<v-toolbar-items>
				<v-menu offset-y v-if="drives.length">
					<template v-slot:activator="{ on }">
						<v-btn text v-on="on" class="text-none">
							<v-icon>mdi-cloud</v-icon>&nbsp;{{
								currentDrive.text
							}}<v-icon>mdi-menu-down</v-icon>
						</v-btn>
					</template>
					<v-list>
						<v-list-item
							v-for="(item, index) in drives"
							:key="index.id"
							@click="changeDrive(item.value)"
						>
							<v-list-item-title>{{
								item.text
							}}</v-list-item-title>`
						</v-list-item>
					</v-list>
				</v-menu>
			</v-toolbar-items>
			<portal-target name="navbar" slim />
			<v-spacer />
			<v-toolbar-items>
				<v-btn
					text
					class="text-none hidden-sm-and-down"
					tag="a"
					href="https://github.com/maple3142/GDIndex"
					target="_blank"
				>
					<v-icon>mdi-github-circle</v-icon>&nbsp;GitHub</v-btn
				>
			</v-toolbar-items>
		</v-app-bar>
		<v-content>
      <div class="row justify-center" style="padding: 20px">
        <div>
          <p>划水交流群933363822，目前录制了1300+个V的B站/油管/t台直播（2434/holo/vr/upd8/774等都录了，b站粉大于2w5的也都录了）</p>
          <p>当天的录播文件夹为vtbrec，每天早上八点前结束的直播算为昨天的直播，请去对应日期的文件夹中寻找</p>
          <p>找不到自家的录播可以加群问一下~ 下载较大的录播档或者希望预览的话推荐加群了解另一种访问方式，字幕组想要加速下载请加群联系群主~~</p>
        </div>
      </div>
      <router-view />
    </v-content>
		<LoginDialog :cond="showAuthInput" />
	</v-app>
</template>
<script>
import api from './api'
import LoginDialog from './components/LoginDialog.vue'

export default {
	props: {
		title: String
	},
	data() {
		return {
			drives: [],
			value: {},
			showAuthInput: false
		}
	},
	computed: {
		currentDrive() {
			const id = this.$route.query.rootId || window.props.default_root_id
			return this.drives.find(d => d.value === id)
		}
	},
	async created() {
		const ok =
			new URL(window.props.api).hostname === location.hostname ||
			(await api
				.get(window.props.api)
				.then(() => true)
				.catch(err => {
					if (err.response.status === 401) {
						this.showAuthInput = true
						return false
					}
				}))
		if (!ok) return

		const { drives } = await api.get('/~_~_gdindex/drives').json()
		//this.drives = [{ text: this.$t('mainDrive'), value: 'root' }].concat(
    this.drives = [].concat(
			drives.map(d => ({
				value: d.id,
				text: d.name
			}))
		)
	},
	methods: {
		changeDrive(drive) {
			const rootId =
				drive !== window.props.default_root_id ? drive : undefined
			const dest = { path: '/', query: { rootId } }
			if (
				dest.path === this.$route.path &&
				dest.query.rootId === this.$route.query.rootId
			) {
				return // vue-router forbid going to same location
			}
			this.$router.push({ path: '/', query: { rootId } })
		}
	},
	components: { LoginDialog }
}
</script>
