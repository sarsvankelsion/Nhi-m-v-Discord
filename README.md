# Hướng dẫn hoàn thành Quest Discord (Bản dịch tiếng Việt)

> **Lưu ý**
>
> Cách này **không hoạt động trên trình duyệt** đối với các quest yêu cầu **chơi game**.
> Hãy sử dụng **ứng dụng Discord trên máy tính** để hoàn thành các quest này.

---

## Cách sử dụng

1. Chấp nhận một quest tại **Discover → Quests**.
2. Nhấn **Ctrl + Shift + I** để mở **DevTools**.
3. Chuyển sang tab **Console**.
4. Dán **đoạn script bên dưới (giữ nguyên, không chỉnh sửa)** và nhấn **Enter**.

   * Nếu không thể dán, hãy gõ `allow pasting` rồi nhấn Enter, sau đó dán lại.
5. Làm theo các hướng dẫn được **in ra trong Console**, tùy theo loại quest:

   * Nếu quest yêu cầu **“play” (chơi game)** hoặc **xem video** → chỉ cần **chờ**, không cần thao tác gì thêm.
   * Nếu quest yêu cầu **“stream” (phát trực tiếp)** → tham gia một **voice chat** với bạn bè hoặc tài khoản phụ và **stream bất kỳ cửa sổ nào**.
6. Chờ một lúc để quest được hoàn thành.
7. Sau khi hoàn tất, bạn có thể **nhận phần thưởng**.

---

## Theo dõi tiến trình

Bạn có thể kiểm tra tiến trình quest bằng:

* Các dòng **`Quest progress:`** hiển thị trong tab **Console**.
* Thanh tiến trình trong tab **Quests** của Discord.

---

## Script (giữ nguyên – KHÔNG chỉnh sửa)

```js
delete window.$;
let wpRequire = webpackChunkdiscord_app.push([[Symbol()], {}, r => r]);
webpackChunkdiscord_app.pop();


let ApplicationStreamingStore = Object.values(wpRequire.c).find(x => x?.exports?.Z?.__proto__?.getStreamerActiveStreamMetadata)?.exports?.Z;
let RunningGameStore, QuestsStore, ChannelStore, GuildChannelStore, FluxDispatcher, api
if(!ApplicationStreamingStore) {
	ApplicationStreamingStore = Object.values(wpRequire.c).find(x => x?.exports?.A?.__proto__?.getStreamerActiveStreamMetadata).exports.A;
	RunningGameStore = Object.values(wpRequire.c).find(x => x?.exports?.Ay?.getRunningGames).exports.Ay;
	QuestsStore = Object.values(wpRequire.c).find(x => x?.exports?.A?.__proto__?.getQuest).exports.A;
	ChannelStore = Object.values(wpRequire.c).find(x => x?.exports?.A?.__proto__?.getAllThreadsForParent).exports.A;
	GuildChannelStore = Object.values(wpRequire.c).find(x => x?.exports?.Ay?.getSFWDefaultChannel).exports.Ay;
	FluxDispatcher = Object.values(wpRequire.c).find(x => x?.exports?.h?.__proto__?.flushWaitQueue).exports.h;
	api = Object.values(wpRequire.c).find(x => x?.exports?.Bo?.get).exports.Bo;
} else {
	RunningGameStore = Object.values(wpRequire.c).find(x => x?.exports?.ZP?.getRunningGames).exports.ZP;
	QuestsStore = Object.values(wpRequire.c).find(x => x?.exports?.Z?.__proto__?.getQuest).exports.Z;
	ChannelStore = Object.values(wpRequire.c).find(x => x?.exports?.Z?.__proto__?.getAllThreadsForParent).exports.Z;
	GuildChannelStore = Object.values(wpRequire.c).find(x => x?.exports?.ZP?.getSFWDefaultChannel).exports.ZP;
	FluxDispatcher = Object.values(wpRequire.c).find(x => x?.exports?.Z?.__proto__?.flushWaitQueue).exports.Z;
	api = Object.values(wpRequire.c).find(x => x?.exports?.tn?.get).exports.tn;	
}

const supportedTasks = ["WATCH_VIDEO", "PLAY_ON_DESKTOP", "STREAM_ON_DESKTOP", "PLAY_ACTIVITY", "WATCH_VIDEO_ON_MOBILE"]
let quests = [...QuestsStore.quests.values()].filter(x => x.userStatus?.enrolledAt && !x.userStatus?.completedAt && new Date(x.config.expiresAt).getTime() > Date.now() && supportedTasks.find(y => Object.keys((x.config.taskConfig ?? x.config.taskConfigV2).tasks).includes(y)))
let isApp = typeof DiscordNative !== "undefined"
if(quests.length === 0) {
	console.log("You don't have any uncompleted quests!")
} else {
	let doJob = function() {
		const quest = quests.pop()
		if(!quest) return

		const pid = Math.floor(Math.random() * 30000) + 1000
		
		const applicationId = quest.config.application.id
		const applicationName = quest.config.application.name
		const questName = quest.config.messages.questName
		const taskConfig = quest.config.taskConfig ?? quest.config.taskConfigV2
		const taskName = supportedTasks.find(x => taskConfig.tasks[x] != null)
		const secondsNeeded = taskConfig.tasks[taskName].target
		let secondsDone = quest.userStatus?.progress?.[taskName]?.value ?? 0

		if(taskName === "WATCH_VIDEO" || taskName === "WATCH_VIDEO_ON_MOBILE") {
			const maxFuture = 10, speed = 7, interval = 1
			const enrolledAt = new Date(quest.userStatus.enrolledAt).getTime()
			let completed = false
			let fn = async () => {			
				while(true) {
					const maxAllowed = Math.floor((Date.now() - enrolledAt)/1000) + maxFuture
					const diff = maxAllowed - secondsDone
					const timestamp = secondsDone + speed
					if(diff >= speed) {
						const res = await api.post({url: `/quests/${quest.id}/video-progress`, body: {timestamp: Math.min(secondsNeeded, timestamp + Math.random())}})
						completed = res.body.completed_at != null
						secondsDone = Math.min(secondsNeeded, timestamp)
					}
					
					if(timestamp >= secondsNeeded) {
						break
					}
					await new Promise(resolve => setTimeout(resolve, interval * 1000))
				}
				if(!completed) {
					await api.post({url: `/quests/${quest.id}/video-progress`, body: {timestamp: secondsNeeded}})
				}
				console.log("Quest completed!")
				doJob()
			}
			fn()
			console.log(`Spoofing video for ${questName}.`)
		} else if(taskName === "PLAY_ON_DESKTOP") {
			if(!isApp) {
				console.log("This no longer works in browser for non-video quests. Use the discord desktop app to complete the", questName, "quest!")
			} else {
				api.get({url: `/applications/public?application_ids=${applicationId}`}).then(res => {
					const appData = res.body[0]
					const exeName = appData.executables.find(x => x.os === "win32").name.replace(">","")
					
					const fakeGame = {
						cmdLine: `C:\\Program Files\\${appData.name}\\${exeName}`,
						exeName,
						exePath: `c:/program files/${appData.name.toLowerCase()}/${exeName}`,
						hidden: false,
						isLauncher: false,
						id: applicationId,
						name: appData.name,
						pid: pid,
						pidPath: [pid],
						processName: appData.name,
						start: Date.now(),
					}
					const realGames = RunningGameStore.getRunningGames()
					const fakeGames = [fakeGame]
					const realGetRunningGames = RunningGameStore.getRunningGames
					const realGetGameForPID = RunningGameStore.getGameForPID
					RunningGameStore.getRunningGames = () => fakeGames
					RunningGameStore.getGameForPID = (pid) => fakeGames.find(x => x.pid === pid)
					FluxDispatcher.dispatch({type: "RUNNING_GAMES_CHANGE", removed: realGames, added: [fakeGame], games: fakeGames})
					
					let fn = data => {
						let progress = quest.config.configVersion === 1 ? data.userStatus.streamProgressSeconds : Math.floor(data.userStatus.progress.PLAY_ON_DESKTOP.value)
						console.log(`Quest progress: ${progress}/${secondsNeeded}`)
						
						if(progress >= secondsNeeded) {
							console.log("Quest completed!")
							
							RunningGameStore.getRunningGames = realGetRunningGames
							RunningGameStore.getGameForPID = realGetGameForPID
							FluxDispatcher.dispatch({type: "RUNNING_GAMES_CHANGE", removed: [fakeGame], added: [], games: []})
							FluxDispatcher.unsubscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
							
							doJob()
						}
					}
					FluxDispatcher.subscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
					
					console.log(`Spoofed your game to ${applicationName}. Wait for ${Math.ceil((secondsNeeded - secondsDone) / 60)} more minutes.`)
				})
			}
		} else if(taskName === "STREAM_ON_DESKTOP") {
			if(!isApp) {
				console.log("This no longer works in browser for non-video quests. Use the discord desktop app to complete the", questName, "quest!")
			} else {
				let realFunc = ApplicationStreamingStore.getStreamerActiveStreamMetadata
				ApplicationStreamingStore.getStreamerActiveStreamMetadata = () => ({
					id: applicationId,
					pid,
					sourceName: null
				})
				
				let fn = data => {
					let progress = quest.config.configVersion === 1 ? data.userStatus.streamProgressSeconds : Math.floor(data.userStatus.progress.STREAM_ON_DESKTOP.value)
					console.log(`Quest progress: ${progress}/${secondsNeeded}`)
					
					if(progress >= secondsNeeded) {
						console.log("Quest completed!")
						
						ApplicationStreamingStore.getStreamerActiveStreamMetadata = realFunc
						FluxDispatcher.unsubscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
						
						doJob()
					}
				}
				FluxDispatcher.subscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
				
				console.log(`Spoofed your stream to ${applicationName}. Stream any window in vc for ${Math.ceil((secondsNeeded - secondsDone) / 60)} more minutes.`)
				console.log("Remember that you need at least 1 other person to be in the vc!")
			}
		} else if(taskName === "PLAY_ACTIVITY") {
			const channelId = ChannelStore.getSortedPrivateChannels()[0]?.id ?? Object.values(GuildChannelStore.getAllGuilds()).find(x => x != null && x.VOCAL.length > 0).VOCAL[0].channel.id
			const streamKey = `call:${channelId}:1`
			
			let fn = async () => {
				console.log("Completing quest", questName, "-", quest.config.messages.questName)
				
				while(true) {
					const res = await api.post({url: `/quests/${quest.id}/heartbeat`, body: {stream_key: streamKey, terminal: false}})
					const progress = res.body.progress.PLAY_ACTIVITY.value
					console.log(`Quest progress: ${progress}/${secondsNeeded}`)
					
					await new Promise(resolve => setTimeout(resolve, 20 * 1000))
					
					if(progress >= secondsNeeded) {
						await api.post({url: `/quests/${quest.id}/heartbeat`, body: {stream_key: streamKey, terminal: true}})
						break
					}
				}
				
				console.log("Quest completed!")
				doJob()
			}
			fn()
		}
	}
	doJob()
}
```

---

## Câu hỏi thường gặp (FAQ)

### ❓ Có thể bị ban khi dùng cách này không?

**Trả lời:** Luôn tồn tại rủi ro. Tuy nhiên, cho đến nay chưa có báo cáo nào về việc bị ban vì cách này hoặc các phương pháp tương tự (ví dụ: client mod).

---

### ❓ Ctrl + Shift + I không hoạt động

**Trả lời:**

* Cài đặt **Discord PTB** (Public Test Build), hoặc
* Làm theo hướng dẫn để bật **DevTools** trên bản Discord stable.

---

### ❓ Ctrl + Shift + I lại chụp màn hình

**Trả lời:** Hãy tắt phím tắt này trong **AMD Radeon Software**.

---

### ❓ Gặp lỗi cú pháp hoặc lỗi “unexpected token”

**Trả lời:**

* Đảm bảo trình duyệt **không tự động dịch trang web**.
* Tắt các tiện ích dịch tự động, sau đó sao chép và dán lại script.

---

### ❓ Dùng Vesktop nhưng bị nhận là đang dùng trình duyệt

**Trả lời:** Vesktop chỉ là một trình duyệt bọc giao diện, **không phải client desktop thật**. Hãy tải **ứng dụng Discord desktop chính thức**.

---

### ❓ Gặp lỗi khác

**Trả lời:** Kiểm tra lại các bước đã làm và đảm bảo bạn **sao chép đúng script** và thực hiện đầy đủ hướng dẫn.

---

### ❓ Có thể hoàn thành quest đã hết hạn không?

**Trả lời:** Không. Không có cách nào để hoàn thành quest đã hết hạn.

---

### ❓ Có thể tự động chấp nhận quest hoặc nhận thưởng không?

**Trả lời:** Không. Các thao tác này có thể kích hoạt **captcha**, nên không nên tự động hóa. Bạn hãy tự bấm hai lần đó bằng tay.

---

### ❓ Có thể làm thành plugin Vencord không?

**Trả lời:** Không. Script này thường xuyên cần cập nhật theo thay đổi của Discord, trong khi chu kỳ cập nhật và kiểm duyệt của Vencord quá chậm. Một số bản fork của Vencord có tích hợp sẵn, nếu bạn thực sự cần.

---

### ❓ Có thể đưa script lên repo và dùng lệnh fetch() một dòng không?

**Trả lời:** Không. Việc này có thể gây **rủi ro bảo mật**, vì script gốc có thể bị thay đổi thành mã độc mà người dùng không hề hay biết.

---

*Ghi chú: Không nên đăng các phiên bản “đã sửa” hoặc “đã cải tiến” của script trong phần bình luận, vì có thể gây nhầm lẫn và làm tăng rủi ro cho người khác.*
