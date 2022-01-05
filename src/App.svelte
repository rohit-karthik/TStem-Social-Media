<script>
	let posts;
	let name = "";
	let desc = "";

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
		const res = await supabase.from("posts").select("*");
		posts = res.data;
		return res.data;
	}

	async function addPost() {
		await supabase.from("posts").insert([
			{
				name: name,
				description: desc,
				email: email,
			},
		]);
		name = "";
		desc = "";
	}

	async function deletePost(postI, postN, postD, postE) {
		if (email.toLowerCase() == postE.toLowerCase()) {
			errorMsg = "";
			deleting = true;
			await supabase.from("posts").delete().match({
				name: postN,
				description: postD,
				id: postI,
			});
			await getData();
			deleting = false;
		} else {
			errorMsg = "You didn't create this post!";
			setTimeout(() => {
				errorMsg = "";
			}, 1500);
		}
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
		}
		else {
			signedIn = false;
			location.reload()
			//localStorage.signedIn = false
		}
	}

	supabase
		.from("posts")
		.on("*", (res) => {
			console.log(res);

			posts = [...posts, res.new];
			console.log(posts);
		})
		.subscribe();
</script>

{#if userSess !== null}
	{#await getData()}
		<p>Fetching</p>
	{:then}
		{#if !deleting}
		<p>{userSess}</p>
			<p style="color: red;">{errorMsg}</p>
			{#each posts.reverse() as post}
				<h1>{post.name}</h1>
				<p>{post.description}</p>
				<button
					on:click={() => {
						deletePost(
							post.id,
							post.name,
							post.description,
							post.email
						);
					}}>Delete this post</button
				>
				<p>This is from: {post.email}</p>
				<hr />
			{/each}
			<button on:click={addPost} class="bg-emerald-400 p-2 m-1"
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
			<button
				class="border-2 border-emerald-400 p-2 m-1 rounded-sm"
				on:click={async () => {
					await signOut();
				}}>Sign Out</button
			>
		{:else}
			<h1>Deleting post...</h1>
		{/if}
	{/await}
{:else}
	<p>{userSess}</p>
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
