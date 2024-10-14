<script lang="ts">
	let video: HTMLVideoElement;
	let canvas: HTMLCanvasElement;
	let photo: HTMLImageElement;
	let startbutton: HTMLButtonElement;
	let mediaDevices: MediaDeviceInfo[] = $state([]);
	let deviceId: string | null = $state(null);

	function acticateWebcam() {
		navigator.mediaDevices
			.getUserMedia({ video: deviceId ? { deviceId: deviceId } : true, audio: false })
			.then((stream) => {
				video.srcObject = stream;
				video.play();
			})
			.catch((err) => {
				console.log('An error occurred: ' + err);
			});
	}

	$effect(() => {
		navigator.mediaDevices.enumerateDevices().then((devices) => {
			for (const device of devices) {
				if (device.kind === 'videoinput' && deviceId === null) {
					deviceId = device.deviceId;
					break;
				}
			}
			mediaDevices = devices.filter((d) => d.kind === 'videoinput');
		});

		startbutton.addEventListener(
			'click',
			(ev) => {
				takepicture();
				ev.preventDefault();
			},
			false
		);

		const takepicture = () => {
			const context = canvas.getContext('2d');
			if (context && video.videoWidth && video.videoHeight) {
				canvas.width = video.videoWidth;
				canvas.height = video.videoHeight;
				context.drawImage(video, 0, 0, video.videoWidth, video.videoHeight);

				const data = canvas.toDataURL('image/png');
				photo.setAttribute('src', data);
			} else {
				clearphoto();
			}
		};

		const clearphoto = () => {
			const context = canvas.getContext('2d');
			if (context) {
				context.fillStyle = '#AAA';
				context.fillRect(0, 0, canvas.width, canvas.height);
			}

			const data = canvas.toDataURL('image/png');
			photo.setAttribute('src', data);
		};
	});
</script>

<select bind:value={deviceId}>
	{#each mediaDevices as d}
		<option value={d.deviceId}>{d.label}</option>
	{/each}
</select>
<button onclick={acticateWebcam}>activate webcam</button>
<br />
<video bind:this={video}></video>
<img bind:this={photo} />
<canvas bind:this={canvas}></canvas>
<br />
<button bind:this={startbutton}>capture</button>

<style>
	video {
		width: 20%;
		height: auto;
	}
	img {
		width: 10%;
		height: auto;
	}
	canvas {
		display: none;
	}
</style>
