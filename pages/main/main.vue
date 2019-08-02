<template>
	<view class="content">
		<view v-if="hasLogin" class="hello">
			<view class="title">
				您好 {{userName}}，您已成功登录。
			</view>
			<view class="ul">

			</view>
		</view>
		<view v-if="!hasLogin" class="hello">
			<view class="title">
				您好,总共联系人{{ allNum }},可读联系人{{ canNum }}
			</view>
			<view class="ul">

			</view>
		</view>
	</view>
</template>

<script>
	import {
		mapState
	} from 'vuex'

	export default {
		data() {
			return {
				allNum:0,
				canNum:0
			}
		},
		computed: mapState(['forcedLogin', 'hasLogin', 'userName']),
		onLoad() {
			this.getContacts();
			return;
			if (!this.hasLogin) {
				uni.showModal({
					title: '未登录',
					content: '您未登录，需要登录后才能继续',
					/**
					 * 如果需要强制登录，不显示取消按钮
					 */
					showCancel: !this.forcedLogin,
					success: (res) => {
						if (res.confirm) {
							/**
							 * 如果需要强制登录，使用reLaunch方式
							 */
							if (this.forcedLogin) {
								uni.reLaunch({
									url: '../login/login'
								});
							} else {
								uni.navigateTo({
									url: '../login/login'
								});
							}
						}
					}
				});
			}
		},
		methods: {
			//处理联系列表
			dealContactList: function(lists) {
				var contactList = [];
				console.log("总共联系人:"+lists.length)
				this.allNum = lists.length
				let contactNum = 0
				for (var list of lists) {
					// console.log(list)
					let name = list.displayName
					let phoneNumbers = list['phoneNumbers'][0]
					// console.log(typeof(phoneNumbers))
					let phone = typeof(phoneNumbers) !== 'object' || typeof(phoneNumbers) === 'undefined' || phoneNumbers == [] ? '' :
						phoneNumbers.value
						if(phone != '')
						{
							++contactNum
						}
					contactList.push({
						"name": name,
						"phone": phone
					})
				}
				console.log("可读联系人:"+contactNum)
				this.canNum = contactNum
				return contactList;
			},
			//获取联系人信息
			getContacts: function() {
				var that = this
				//获取通讯录对象
				plus.contacts.getAddressBook(plus.contacts.ADDRESSBOOK_PHONE, function(addressbook) {
					uni.showToast({
						title: '获取通讯录对象成功',
						duration: 2000
					})
					console.log('获取通讯录对象成功')
					console.log(addressbook)
					// 查找联系人
					addressbook.find(["displayName", "phoneNumbers"], function(contacts) {
						uni.showToast({
							title: '获取联系人成功',
							duration: 2000
						})
						console.log('获取联系人成功')
						// console.log(JSON.stringify(contacts))
						uni.showModal({
							title: '联系人信息',
							content: JSON.stringify(that.dealContactList(contacts)),
						});
						that.list = contacts
					}, function() {
						uni.showToast({
							title: '获取联系人失败',
							duration: 2000
						})
					}, {
						multiple: true
					});
				}, function(e) {
					uni.showToast({
						title: '获取通讯录对象失败:' + e.message,
						duration: 2000
					})
				});
			}
		}
	}
</script>

<style>
	.hello {
		display: flex;
		flex: 1;
		flex-direction: column;
	}

	.title {
		color: #8f8f94;
		margin-top: 50upx;
	}

	.ul {
		font-size: 30upx;
		color: #8f8f94;
		margin-top: 50upx;
	}

	.ul>view {
		line-height: 50upx;
	}
</style>
