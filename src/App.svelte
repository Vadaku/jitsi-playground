<script>
	import JitsiMeetJS from "@lyno/lib-jitsi-meet";

	const connOptions = {
		hosts: {
			domain: "meet.jit.si",
			muc: "conference.meet.jit.si"
		},
		bosh: "https://meet.jit.si/http-bind"
	}

	let tracksArr = [];
	
	JitsiMeetJS.setLogLevel(JitsiMeetJS.logLevels.ERROR)

	const initOptions = {}

	const onTrackAdded = () => {
		console.log("TRACK ADDED")
	}

	const onConnectionSuccess = () => {
		let roomURL = "https://" + connOptions.hosts.domain + "/21e8call"
		console.log("Connection Successful")
		let room = connection.initJitsiConference("21e8call", {})
		room.on(JitsiMeetJS.events.conference.TRACK_ADDED, onTrackAdded);
		room.on(JitsiMeetJS.events.conference.CONFERENCE_JOINED, async () => {
    		const tracks = await JitsiMeetJS.createLocalTracks({devices: ['video', 'audio']})
			tracks.forEach(track => {
				room.addTrack(track)
				tracksArr = [...tracksArr, track]
			});
		})
		room.join(null, null)
		window.open(roomURL, "POPUP", "height=800,width=800,left=100,top=100")
	}

	const onDisconnect = () => {
		console.log("Disconnected")
	}

	JitsiMeetJS.init(initOptions)
	
	let connection = new JitsiMeetJS.JitsiConnection(null, null, connOptions);
	connection.addEventListener(JitsiMeetJS.events.connection.CONNECTION_ESTABLISHED, onConnectionSuccess);
	connection.addEventListener(JitsiMeetJS.events.connection.CONNECTION_DISCONNECTED, onDisconnect);

	
	// JitsiMeetJS.createLocalTracks().then(onLocalTracks);
</script>

<main>
	<h1>JITSI 21e8</h1>
	<button on:click={() => connection.connect({})}>Join Room</button>
	<button on:click={() => connection.disconnect({})}>Leave Room</button>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
