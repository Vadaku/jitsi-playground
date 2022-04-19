<script>
	import JitsiMeetJS from "@lyno/lib-jitsi-meet";

    export let data;

	const connOptions = {
		hosts: {
			domain: "meet.jit.si",
			muc: "conference.meet.jit.si"
		},
		serviceUrl: "https://meet.jit.si/http-bind"
	}

    let room = null;
	let localTracks = [];
	let remoteTracks = [];
    let showVideo = false;

	JitsiMeetJS.setLogLevel(JitsiMeetJS.logLevels.ERROR)

	const initOptions = {}

	const onRemoteTrack = (track) => {
		console.log("Track being added")
		if (track.isLocal()) {
			return;
		}
		track.forEach((track, i) => {
			room.addTrack(track)
			if (track.getType() == "video") {
				let vidContainer = document.getElementById("remote-con")
				let vid = document.createElement("video")
				vid.setAttribute(`id`, `remote-video${i}`)
				vid.setAttribute(`height`, `500`)
				vid.setAttribute(`width`, `500`)
				vid.setAttribute(`autoplay`, 1)
				vidContainer.append(vid)
				track.attach(vid);
			} else {
				let audContainer = document.getElementById("remote-con")
				let aud = document.createElement("audio")
				aud.setAttribute(`id`, `remote-audio${i}`)
				aud.setAttribute(`autoplay`, 1)
				audContainer.append(aud)
				track.attach(aud);
			}
		});
	}

    const onLocalTracks = (tracks) => {
		console.log(tracks)
		tracks.forEach((track, i) => {
			// room.addTrack(track)
			if (track.getType() == "video") {
                    let vidContainer = document.getElementById("con")
					let vid = document.createElement("video")
					vid.setAttribute(`id`, `local-video${i}`)
					vid.setAttribute(`autoplay`, 1)
					vid.setAttribute(`height`, `500`)
					vid.setAttribute(`width`, `500`)
					vidContainer.append(vid)
					track.attach(vid);
				} else {
					let audContainer = document.getElementById("con")
					let aud = document.createElement("audio")
					aud.setAttribute(`id`, `local-audio${i}`)
					aud.setAttribute(`autoplay`, 1)
					audContainer.append(aud)
					track.attach(aud);
				}
		})
    }

	const onConnectionSuccess = () => {
		let roomURL = "https://" + connOptions.hosts.domain + "/" + data
		console.log("Connection Successful")
		room = connection.initJitsiConference("21e8call", {})
		room.on(JitsiMeetJS.events.conference.TRACK_ADDED, onRemoteTrack);
		room.on(JitsiMeetJS.events.conference.CONFERENCE_JOINED, () => {
    		console.log("Conference Joined")
		})
		room.on(JitsiMeetJS.events.conference.USER_JOINED, () => {
			console.log("User Joined")
		})
		room.join(null, null)
		// window.open(roomURL, "POPUP", "fullscreen=yes")
	}

	const onDisconnect = () => {
		console.log("Jitsi Disconnected")
		connection.removeEventListener(JitsiMeetJS.events.connection.CONNECTION_ESTABLISHED, onConnectionSuccess)
		connection.removeEventListener(JitsiMeetJS.events.connection.CONNECTION_FAILED, () => {
			console.log("Jitsi Disconnected")
		})
		connection.removeEventListener(JitsiMeetJS.events.connection.CONNECTION_DISCONNECTED, onDisconnect)
	}

	JitsiMeetJS.init(initOptions)
	
    connOptions.serviceUrl = connOptions.serviceUrl + "?room=21e8call"
	let connection = new JitsiMeetJS.JitsiConnection(null, null, connOptions);
	connection.addEventListener(JitsiMeetJS.events.connection.CONNECTION_ESTABLISHED, onConnectionSuccess);
	connection.addEventListener(JitsiMeetJS.events.connection.CONNECTION_FAILED, () => {
		console.log("Connection to Jitsi failed")
	});
	connection.addEventListener(JitsiMeetJS.events.connection.CONNECTION_DISCONNECTED, onDisconnect);
	connection.connect()
	
	// JitsiMeetJS.createLocalTracks({devices: ["audio", "video"]}).then(onLocalTracks);
</script>

<main>
	<h1>JITSI 21e8</h1>
	Local Video
	<div id="con">
		<!-- svelte-ignore a11y-media-has-caption -->
        	<!-- <video autoplay="1" id="vid" width="500" height="500"></video>
			<audio autoplay="1" muted="true" id="aud"></audio> -->
	</div>
	Remote Video
	<div id="remote-con">
		<!-- svelte-ignore a11y-media-has-caption -->
        	<!-- <video autoplay="1" id="vid" width="500" height="500"></video>
			<audio autoplay="1" muted="true" id="aud"></audio> -->
	</div>
	<button on:click={() => {connection.connect();}}>Join Room</button>
	<button on:click={() => {connection.disconnect({}); showVideo = false;}}>Leave Room</button>
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
		font-size: 3em;
		font-weight: 100;
	}

	#con {
		display: flex;
		border: 1px solid black;
		height: calc(100vh - 30vh);
		margin-bottom: 1em;
	}

	#remote-con {
		display: flex;
		border: 1px solid black;
		height: calc(100vh - 30vh);
		margin-bottom: 1em;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
