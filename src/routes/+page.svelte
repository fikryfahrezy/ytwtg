<script lang="ts">
	type Chat = {
		type: 'chat';
		message: string;
	};

	type PlayEvent = {
		type: 'playevent';
		is_playing: boolean;
	};

	type Message = Chat | PlayEvent;

	import type { DOMAttributes } from 'svelte/elements';
	import SveltPlayer from 'svelte-player';
	import { PUBLIC_SOCKET_URL } from '$env/static/public';

	let socket: WebSocket;

	let playing: boolean = $state(false);

	let chats: Chat[] = $state([]);

	function onSendMessage(chat: Message) {
		socket.send(JSON.stringify(chat));
	}

	const handleChatSubmit: DOMAttributes<HTMLFormElement>['on:submit'] = (e) => {
		const formValues = Object.fromEntries(new FormData(e.currentTarget)) as Chat;
		onSendMessage({ ...formValues, type: 'chat' });
		e.currentTarget.reset();
	};

	function onPlayerPlay(e: CustomEvent) {
		onSendMessage({ type: 'playevent', is_playing: true });
	}

	function onPlayerPause(e: CustomEvent) {
		onSendMessage({ type: 'playevent', is_playing: false });
	}

	$effect(() => {
		// Create WebSocket connection.
		socket = new WebSocket(PUBLIC_SOCKET_URL);

		// Connection opened
		socket.addEventListener('open', (e) => {
			const firstMessage: Message = {
				type: 'chat',
				message: 'Welcome!'
			};
			onSendMessage(firstMessage);
		});

		socket.addEventListener('message', (event) => {
			const message = JSON.parse(event.data) as Message;
			switch (message.type) {
				case 'playevent':
					playing = message.is_playing;
					break;
				default:
					chats = [...chats, message];
			}
		});
	});
</script>

<main class="root">
	<div class="player-container">
		<div class="video-container">
			<SveltPlayer
				url="https://youtu.be/oUFJJNQGwhk"
				{playing}
				on:play={onPlayerPlay}
				on:pause={onPlayerPause}
			/>
		</div>
	</div>
	<div class="chat-container">
		<div class="chat-list">
			{#each chats as chat}
				<p>{chat.message}</p>
			{/each}
		</div>
		<form class="chat-form" on:submit|preventDefault={handleChatSubmit}>
			<input class="chat-input" name="message" placeholder="Write messege here..." />
		</form>
	</div>
</main>

<style>
	.root {
		display: flex;
		height: 100vh;
	}

	.player-container {
		flex: 2;
	}

	.video-container {
		max-width: 1280px;
		width: 100%;
		aspect-ratio: 16/9;
	}

	.chat-container {
		display: flex;
		border: 1px solid grey;
		flex-direction: column;
		flex: 1;
	}

	.chat-list {
		flex: 1;
		overflow: scroll;
	}

	.chat-input,
	.chat-input {
		width: 100%;
		max-height: 150px;
	}
</style>
