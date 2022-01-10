<script>
	let posts;
	let name = "";
	let desc = "";

	let chatOrTodo = "chat";

	let newChannel = "";
	let channels = [];
	let activeChannel = "null";
	let newEmail = "";

	let completedTodos = [];
	let todos = [];

	let signedIn = false;

	let errorMsg = "";

	let email = "";
	let pass = "";

	let deleting = false;

	let opened = false;

	import { createClient } from "@supabase/supabase-js";

	export const supabase = createClient(
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
				.eq("channel", activeChannel)
				.order("id");
			posts = res.data;
			todos = [];
			const res2 = await supabase
				.from("todos")
				.select("*")
				.eq("channel", activeChannel)
				.eq("completed", false)
				.order("id");

			todos = res2.data;
			completedTodos = [];
			const res3 = await supabase
				.from("todos")
				.select("*")
				.eq("channel", "null")
				.eq("completed", true)
				.order("id");
			completedTodos = res3.data;
		} else {
			const res = await supabase
				.from("posts")
				.select("*")
				.eq("channel", "null")
				.order("id");
			posts = res.data;
			todos = [];
			const res2 = await supabase
				.from("todos")
				.select("*")
				.eq("channel", "null")
				.eq("completed", false)
				.order("id");
			todos = res2.data;
			completedTodos = [];
			const res3 = await supabase
				.from("todos")
				.select("*")
				.eq("channel", "null")
				.eq("completed", true)
				.order("id");

			completedTodos = res3.data;
		}
	}

	async function updateTodo(newValue, todoName) {
		await supabase
			.from("todos")
			.update({
				completed: newValue,
			})
			.eq("name", todoName);
	}

	async function addPost(channelName) {
		if (name == "") {
			errorMsg = "Please enter a post name!";
			setTimeout(() => {
				errorMsg = "";
			}, 1500);
		} else {
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
	}

	async function addTodo(channelName) {
		if (name == "") {
			errorMsg = "Please enter a post name!";
			setTimeout(() => {
				errorMsg = "";
			}, 1500);
		} else {
			await supabase.from("todos").insert([
				{
					name: name,
					completed: false,
					channel: channelName,
					email: userSess.email,
				},
			]);
			name = "";
		}
	}

	async function addChannel() {
		if (newChannel == "") {
			errorMsg = "Please enter a channel name!";
			setTimeout(() => {
				errorMsg = "";
			}, 1500);
		} else {
			await supabase.from("channels").insert([
				{
					name: newChannel,
					access: userSess.email,
				},
			]);
		}
	}

	async function addPerson() {
		console.log(activeChannel);
		const res = await supabase
			.from("channels")
			.select("*")
			.eq("name", activeChannel);
		console.log(res.data);

		const newStr = res.data[0].access.concat(", ", newEmail);
		console.log(newStr);

		await supabase
			.from("channels")
			.update({
				access: newStr,
			})
			.eq("name", activeChannel);
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

	async function deleteTodo(todoI, todoN) {
		errorMsg = "";
		deleting = true;
		await supabase.from("todos").delete().match({
			name: todoN,
			id: todoI,
		});
		await getData();
		deleting = false;
	}

	async function deleteChannel() {
		errorMsg = "";
		deleting = true;
		await supabase.from("channels").delete().match({
			name: activeChannel,
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

	supabase
		.from("todos")
		.on("*", (res) => {
			console.log(res);

			getData();
			//location.reload();
			console.log(todos);
		})
		.subscribe();

	function openSidebar() {
		if (opened) {
			document.getElementById("sidebar").style.width = "0%";
			document.getElementById("mainContent").style.width = "100%";
			document.getElementById("switchButton").style.right = "3rem";
			document.getElementById("infoButton").style.right = "0.25rem";
			opened = false;
		} else {
			document.getElementById("sidebar").style.width = "16.666667%";
			document.getElementById("mainContent").style.width = "66.666667%";
			document.getElementById("switchButton").style.right = "18.75rem";
			document.getElementById("infoButton").style.right = "16rem";
			opened = true;
		}
	}

	function handleKeydown(event) {
		if (event.key == "Enter") {
			addPost(activeChannel);
		}
	}

	function handleKeydownTodo(event) {
		if (event.key == "Enter") {
			addTodo(activeChannel);
		}
	}
</script>

<div class="flex overflow-auto flex-col-reverse h-screen">
	{#if userSess !== null}
		{#await getData()}
			<div class="flex justify-center items-center h-screen">
				<p class="text-5xl text-emerald-500">Grabbing data...</p>
			</div>
		{:then}
			{#if !deleting}
				<div class="flex flex-row bg-white">
					<div
						class="w-2/12 fixed left-0 top-0 h-screen text-white"
						style="background-color: #39133D;"
					>
						<p class="m-2 text-lg">
							The Tesla Stem Productivity App
						</p>
						<hr />
						{#each channels as channel}
							{#if activeChannel == channel}
								<button
									class="w-full text-left pl-1 py-1"
									style="background-color: #2F629E;"
									on:click={() => {
										activeChannel = channel;
										getData();
									}}># {channel}</button
								>
							{:else}
								<button
									class="border-emerald-400 w-full text-left pl-1 py-1"
									on:click={() => {
										activeChannel = channel;
										getData();
									}}># {channel}</button
								>
							{/if}
						{/each}
						{#if activeChannel == "null"}
							<button
								class="w-full text-left pl-1 py-1"
								style="background-color: #2F629E;"
								on:click={() => {
									activeChannel = "null";
									getData();
								}}># Public Chat</button
							>
						{:else}
							<button
								class="w-full text-left pl-1 py-1"
								on:click={() => {
									activeChannel = "null";
									getData();
								}}># Public Chat</button
							>
						{/if}
						<button
							class="bg-red-500 text-white p-2 m-1 rounded-md fixed bottom-2 left-2"
							on:click={async () => {
								await signOut();
							}}>Sign Out</button
						>
					</div>
					{#if chatOrTodo == "chat"}
						<div class="ml-64" id="mainContent" style="width: 100%">
							<div class="fixed top-0">
								<div class="bg-white w-screen p-2">
									<p>
										# {activeChannel == "null"
											? "Public Chat"
											: activeChannel}
									</p>
									<button
										id="switchButton"
										class="border-2 border-black p-1 rounded-md fixed top-1 right-1"
										style="right: 3rem; transition: 0.5s;"
										on:click={() => {
											chatOrTodo = "todos";
										}}>Switch to Team Todos</button
									>
									<button
										id="infoButton"
										class="border-2 border-black p-1 rounded-md fixed top-1 right-1"
										style="right: 0.25rem; transition: 0.5s;"
										on:click={openSidebar}>Info</button
									>
								</div>
							</div>
							<div class="pt-8 pb-16">
								{#each posts as post}
									<p class="text-3xl">{post.name}</p>
									<p>{post.description}</p>
									<div class="flex">
										<p>This is from: {post.email}</p>
										{#if userSess.email.toLowerCase() == post.email}
											<button
												class="ml-3 text-red-500"
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
									</div>
									<hr />
								{/each}
							</div>
							<div class="fixed bottom-0">
								<div class="bg-white w-screen">
									<p style="color: red;">{errorMsg}</p>
									<input
										placeholder="Name of Post (required): "
										bind:value={name}
										on:keydown={handleKeydown}
										class="border-2 p-2 m-1 rounded-md w-64"
									/>
									<input
										placeholder="Description of Post (optional): "
										bind:value={desc}
										on:keydown={handleKeydown}
										class="border-2 resize p-2 m-2 rounded-md w-5/12"
									/>
									<button
										on:click={() => {
											addPost(activeChannel);
										}}
										class="bg-emerald-400 p-2 m-1 shadow-xl rounded-md"
										>Post</button
									>
								</div>
							</div>
						</div>
					{:else}
						<div class="ml-64" id="mainContent" style="width: 100%">
							<div class="fixed top-0">
								<div class="bg-white w-screen p-2">
									<p>
										# {activeChannel == "null"
											? "Public Chat"
											: activeChannel}
									</p>
									<button
										id="switchButton"
										class="border-2 border-black p-1 rounded-md fixed top-1 right-1"
										style="right: 3rem; transition: 0.5s;"
										on:click={() => {
											chatOrTodo = "chat";
										}}>Switch to Team Chat</button
									>
									<button
										id="infoButton"
										class="border-2 border-black p-1 rounded-md fixed top-1 right-1"
										style="right: 0.25rem; transition: 0.5s;"
										on:click={openSidebar}>Info</button
									>
								</div>
							</div>
							<div class="pt-8 pb-16">
								<p class="text-3xl pb-1">In Progress</p>
								{#each todos as todo}
									<label>
										<input
											type="checkbox"
											on:click={() => {
												updateTodo(true, todo.name);
											}}
										/>
										{todo.name}
										{#if userSess.email.toLowerCase() == todo.email}
											<button
												class="ml-3 text-red-500"
												on:click={() => {
													deleteTodo(
														todo.id,
														todo.name
													);
												}}>Delete this todo</button
											>
										{/if}
										<br />
									</label>
								{/each}
								<p class="text-3xl pb-1">Completed</p>
								{#each completedTodos as todo}
									<label>
										<input
											type="checkbox"
											disabled
											checked
											on:click={() => {
												updateTodo(false, todo.name);
											}}
										/>
										{todo.name}
										<button
											class="ml-3 text-emerald-500"
											on:click={() => {
												updateTodo(false, todo.name);
											}}>Make unchecked</button
										>
										{#if userSess.email.toLowerCase() == todo.email}
											<button
												class="ml-3 text-red-500"
												on:click={() => {
													deleteTodo(
														todo.id,
														todo.name
													);
												}}>Delete this todo</button
											>
										{/if}
										<br />
									</label>
								{/each}
							</div>
							<div class="fixed bottom-0">
								<div class="bg-white w-screen">
									<p style="color: red;">{errorMsg}</p>
									<input
										on:keydown={handleKeydownTodo}
										placeholder="Name of Todo (required): "
										bind:value={name}
										class="border-2 p-2 m-1 rounded-md w-64"
									/>
									<button
										on:click={() => {
											addTodo(activeChannel);
										}}
										class="bg-emerald-400 p-2 m-1 shadow-xl rounded-md"
										>Create</button
									>
								</div>
							</div>
						</div>
					{/if}
					<div
						class="fixed right-0 top-0 flex flex-col h-screen"
						style="width: 0%; transition: 0.5s;"
						id="sidebar"
					>
						{#if activeChannel != "null"}
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
						{/if}
						<button
							class="p-2 m-1 rounded-md bg-emerald-400 shadow-lg"
							on:click={async () => {
								await addChannel();
								newChannel = "";
								await getData();
							}}>+ Create a new Channel</button
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
					</div>
				</div>
			{:else}
				<div class="flex justify-center items-center h-screen">
					<p class="text-5xl text-red-500">Deleting item...</p>
				</div>
			{/if}
		{/await}
	{:else}
		<div class="flex flex-col items-center justify-center h-screen">
			<p class="text-5xl m-3">The Tesla Stem Productivity App</p>
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
				class="bg-emerald-400 shadow-sm shadow-emerald-400 p-2 m-1 rounded-md"
				on:click={() => {
					signUp(email, pass);
				}}>Sign up!</button
			>
			<button
				class="border-2 border-emerald-400 p-2 m-1 rounded-md"
				on:click={() => {
					signIn(email, pass);
				}}>Sign In!</button
			>
			<p style="color: red;">{errorMsg}</p>
		</div>
	{/if}
</div>
