<template>
	<div class="pad-top channel-button" @click="leaveChannel" :disabled="!this.currentChannel">{{ isAdmin ? "Delete Channel" : "Leave Channel" }}</div>
</template>

<script>
	import {mapState, mapActions, mapMutations} from "vuex";

	export default {
		computed: {
			...mapState({
				currentChannel: state => state.channels.currentChannel,
				userId: state => state.user.id
			}),

			isAdmin () {
				if (this.currentChannel) {
					return this.currentChannel.admins.includes(this.userId);
				}

				return false;
			}
		},

		methods: {
			leaveChannel () {
				if (this.currentChannel) {
					let method = this.removeChannel;

					if (this.isAdmin) {
						method = this.deleteChannel;
					}

					method(this.currentChannel._id).then(() => {
						return Promise.all([
							this.loadChannels(),
							this.setCurrentChannel(null)
						]);
					}).catch(err => {
						console.error(err);
					});
				}
			},

			loadChannels () {
				this.getChannels({
					criteria: {
						$or: [
							{admins: this.userId},
							{users: this.userId}
						]
					},
					update: true
				}).catch(err => {
					console.error(err);
				});
			},

			...mapActions("channels", ["removeChannel", "deleteChannel", "getChannels"]),
			...mapMutations("channels", ["setCurrentChannel"])
		}
	};
</script>

<style scoped>

</style>
