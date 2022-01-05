<script>
	let posts;
	let name = "";
	let desc = "";
	let signedIn = false;
	let user;
	let errorMsg = "";

	let email = "";
	let pass = "";

	let deleting = false;

	import { createClient } from "@supabase/supabase-js";
	import { get } from "svelte/store";

	const supabase = createClient(
		"https://tymaawbbrmoeljisdgry.supabase.co",
		"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoiYW5vbiIsImlhdCI6MTY0MTI2Nzc0OCwiZXhwIjoxOTU2ODQzNzQ4fQ.wzGimQFfkYZVvDQrT-fG5RTjDZhEBcGbYG6OVyWrNQs"
	);

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
			//user = user;
			signedIn = true;
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
			signedIn = true;
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

{#if signedIn}
	{#await getData()}
		<p>Fetching</p>
	{:then}
		{#if !deleting}
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
			<button on:click={addPost}>Create a post!</button>
			<input placeholder="Name of Post: " bind:value={name} />
			<input placeholder="Description of Post: " bind:value={desc} />
		{:else}
			<h1>Deleting post...</h1>
		{/if}
	{/await}
{:else}
	<input type="email" placeholder="Your Email: " bind:value={email} />
	<input type="password" placeholder="Your Password:  " bind:value={pass} />
	<button
		on:click={() => {
			signUp(email, pass);
		}}>Sign up!</button
	>
	<button
		on:click={() => {
			signIn(email, pass);
		}}>Sign In!</button
	>
	<p style="color: red;">{errorMsg}</p>
{/if}
