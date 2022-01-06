<script>
	let posts;
	let name = "";
	let desc = "";

	let newChannel = "";
	let channels = [];
	let activeChannel = "null";
	let newEmail = ""

	let signedIn = false;

	let errorMsg = "";

	let email = "";
	let pass = "";

	let deleting = false;

	import { createClient } from "@supabase/supabase-js";

	const supabase = createClient(
		"https://tymaawbbrmoeljisdgry.supabase.co",
		"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoiYW5vbiIsImlhdCI6MTY0MTI2Nzc0OCwiZXhwIjoxOTU2ODQzNzQ4fQ.wzGimQFfkYZVvDQrT-fG5RTjDZhEBcGbYG6OVyWrNQs"
	);

	let userSess = supabase.auth.user();

	async function getData() {
		const res2 = await supabase.from("channels").select("*");
		channels = [];
		for (var i in res2.data) {
			if (res2.data[i].access.includes(userSess.email)) {
				channels = [...channels, res2.data[i].name];
			}
		}

		if (activeChannel != "null") {
			const res = await supabase
				.from("posts")
				.select("*")
				.eq("channel", activeChannel);
			posts = res.data;
			return res.data;
		} else {
			const res = await supabase.from("posts").select("*").eq("channel", "null");
			posts = res.data;
			return res.data;
		}
	}

	async function addPost(channelName) {
		await supabase.from("posts").insert([
			{
				name: name,
				description: desc,
				email: userSess.email,
				channel: channelName,
			},
		]);
		name = "";
		desc = "";
	}

	async function addChannel() {
		await supabase.from("channels").insert([
			{
				name: newChannel,
				access: userSess.email,
			},
		]);
	}

	async function addPerson() {
		console.log(activeChannel)
		const res = await supabase.from("channels").select("*").eq("name", activeChannel);
		console.log(res.data)

		const newStr = res.data[0].access.concat(", ", newEmail)
		console.log(newStr)

		await supabase.from("channels").update({
			access: newStr
		}).eq("name", activeChannel);
	}

	async function deletePost(postI, postN, postD, postE) {
		errorMsg = "";
		deleting = true;
		await supabase.from("posts").delete().match({
			name: postN,
			description: postD,
			id: postI,
		});
		await getData();
		deleting = false;
	}

	async function deleteChannel() {
		errorMsg = "";
		deleting = true;
		await supabase.from("channels").delete().match({
			name: activeChannel
		});
		location.reload();
		deleting = false;
	}

	async function signUp(email, password) {
		const { user, session, error } = await supabase.auth.signUp({
			email: email,
			password: password,
		});
		if (error !== null) {
			errorMsg = error.message;
		} else {
			errorMsg = "";
			userSess = user;
			signedIn = true;
			//localStorage.signedIn = true
		}
	}

	async function signIn(email, password) {
		const { user, session, error } = await supabase.auth.signIn({
			email: email,
			password: password,
		});
		if (error !== null) {
			errorMsg = error.message;
		} else {
			errorMsg = "";
			//user = user;
			userSess = user;
			signedIn = true;
			//localStorage.signedIn = true
		}
	}

	async function signOut() {
		const { error } = await supabase.auth.signOut();

		if (error !== null) {
			errorMsg = error.message;
			setTimeout(() => {
				errorMsg = "";
			}, 1500);
		} else {
			signedIn = false;
			location.reload();
			//localStorage.signedIn = false
		}
	}

	supabase
		.from("posts")
		.on("*", (res) => {
			console.log(res);

			getData();
			console.log(posts);
		})
		.subscribe();
</script>

{#if userSess !== null}
	{#await getData()}
		<p>Grabbing posts...</p>
	{:then}
		{#if !deleting}
			<p style="color: red;">{errorMsg}</p>
			{#each posts.reverse() as post}
				<p class="text-3xl">{post.name}</p>
				<p>{post.description}</p>
				{#if userSess.email.toLowerCase() == post.email}
					<button
						class="p-1 m-1 border-2 border-red-500 text-red-500 rounded-md"
						on:click={() => {
							deletePost(
								post.id,
								post.name,
								post.description,
								post.email
							);
						}}>Delete this post</button
					>
				{/if}
				<p>This is from: {post.email}</p>
				<hr />
			{/each}
			<button
				on:click={() => {
					addPost(activeChannel);
				}}
				class="bg-emerald-400 p-2 m-1 shadow-xl rounded-md"
				>Create a post!</button
			>
			<input
				placeholder="Name of Post: "
				bind:value={name}
				class="border-2 p-2 m-1 rounded-md"
			/>
			<input
				placeholder="Description of Post: "
				bind:value={desc}
				class="border-2 p-2 m-1 rounded-md"
			/>
			<hr />
			{#each channels as channel}
				<button
					class="p-1 m-1 rounded-md border-2 border-emerald-500 text-emerald-500"
					on:click={() => {
						activeChannel = channel;
						getData();
					}}>{channel}</button
				>
			{/each}
			<button
				class="p-1 m-1 rounded-md border-2 border-emerald-500 text-emerald-500"
				on:click={() => {
					activeChannel = "null";
					getData();
				}}>Public Chat</button
			>
			<hr />
			<button
				class="p-2 m-1 rounded-md bg-emerald-400 shadow-lg"
				on:click={async () => {
					await addPerson();
					newEmail = "";
					await getData();
				}}>Add a Person to this Channel</button
			>
			<input
				placeholder="Email of Person: "
				bind:value={newEmail}
				class="border-2 p-2 m-1 rounded-md"
			/>
			<hr />
			<button
				class="p-2 m-1 rounded-md bg-emerald-400 shadow-lg"
				on:click={async () => {
					await addChannel();
					newChannel = "";
					await getData();
				}}>Create a Channel</button
			>
			<input
				placeholder="Name of Channel: "
				bind:value={newChannel}
				class="border-2 p-2 m-1 rounded-md"
			/>
			{#if activeChannel != "null"}
				<button
					class="p-2 m-1 rounded-md bg-red-500 shadow-lg"
					on:click={() => {
						deleteChannel();
					}}>Delete this channel</button
				>
			{/if}
			<hr />
			<button
				class="border-2 border-red-500 text-red-500 p-2 m-1 rounded-md"
				on:click={async () => {
					await signOut();
				}}>Sign Out</button
			>
		{:else}
			<h1>Deleting post/channel...</h1>
		{/if}
	{/await}
{:else}
	<input
		type="email"
		placeholder="Your Email: "
		bind:value={email}
		class="border-2 p-2 m-1 rounded-md"
	/>
	<input
		type="password"
		placeholder="Your Password:  "
		bind:value={pass}
		class="border-2 p-2 m-1 rounded-md"
	/>
	<button
		class="bg-emerald-400 shadow-sm shadow-emerald-400 p-2 m-1 rounded-sm"
		on:click={() => {
			signUp(email, pass);
		}}>Sign up!</button
	>
	<button
		class="border-2 border-emerald-400 p-2 m-1 rounded-sm"
		on:click={() => {
			signIn(email, pass);
		}}>Sign In!</button
	>
	<p style="color: red;">{errorMsg}</p>
{/if}
