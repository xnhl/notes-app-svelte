<script>
	import low from "lowdb"
	import { v4 as uuidv4 } from 'uuid'
	import FileSaver from "file-saver"
	import moment from "moment"
	import LocalStorage from "lowdb/adapters/LocalStorage"
	import { onMount } from "svelte"
	import Post from "./components/Post.svelte"
	import Navigation from "./components/Navigation.svelte"
	$: posts = []
	let time = `${moment().format("LL")} ${moment().format("LTS")}`
	$: sortedPosts = () => {
		let sorted = [...posts]
		let reversed = sorted.reverse()
		return reversed
	}
	const search = () => {
		let posts = [...document.getElementById("posts-wrapper").children]
		let searchValue = document.getElementById("search-input").value
		for (let post of posts) {
			let title = post.children[1].textContent.toLowerCase()
			let body = post.children[2].textContent.toLowerCase()
			if (title.indexOf(searchValue) > -1 || body.indexOf(searchValue) > -1) {
				post.classList.remove("hide")
			} else post.classList.add("hide")
		}
	}
	const write = () => {
		const adapter = new LocalStorage('db')
		const db = low(adapter)
		let title = document.getElementById("title-text")
		let titleText = title.value.toString()
		let body = document.getElementById("body-text")
		let bodyText = body.value.toString()
		if (titleText == "" || bodyText == "") {
			console.log("empty")
		} else {
			db.get('posts')
				.unshift({
					id: uuidv4(),
					timestamp: moment().format("LLLL"),
					title: titleText,
					body: bodyText 
				})
				.write()
			let allPosts = db.get('posts').value()
			posts = allPosts
			title.value = ""
			body.value = ""
		}
	}
	const updateList = (event) => {
		const adapter = new LocalStorage('db')
		const db = low(adapter)
		let existing = db.get('posts').value()
		posts = existing
	}
	const addRandom = () => {
		const adapter = new LocalStorage('db')
		const db = low(adapter)
		let list = db.get('posts').value()
		let defaultList = [
			{
				title: "Rerum, velit molestias",
				body: "Lorem ipsum dolor sit, amet consectetur adipisicing elit. Rerum, velit molestias eveniet, animi ipsum praesentium ea voluptatibus necessitatibus eius amet incidunt quod odio vero saepe."
			},
			{
				title: "Laborum itaque maxime saepe",
				body: "Lorem, ipsum dolor sit amet consectetur adipisicing elit. Laborum itaque maxime saepe nihil. Laboriosam, eos neque? Recusandae doloremque praesentium enim vero, asperiores ratione soluta ad."
			},
			{
				title: "Beatae numquam repudiandae recusandae",
				body: "Lorem ipsum dolor sit amet, consectetur adipisicing elit. Beatae numquam repudiandae recusandae libero. Numquam in illo ratione porro unde amet omnis incidunt sed aut quos."
			},
			{
				title: "Nihil provident ipsam,",
				body: "Lorem, ipsum dolor sit amet consectetur adipisicing elit. Nihil provident ipsam, iste, commodi suscipit assumenda dolore sequi architecto veniam totam magni aspernatur est consequuntur eligendi."
			},
			{
				title: "Rerum dolor eaque similique",
				body: "Lorem ipsum dolor sit, amet consectetur adipisicing elit. Rerum dolor eaque similique fugiat? Corrupti et ipsam eligendi quae, ducimus quaerat ea doloremque! Earum, eaque molestias!"
			},
			{
				title: "Possimus illum est ullam",
				body: "Lorem ipsum dolor sit, amet consectetur adipisicing elit. Possimus illum est ullam, dolorum ipsa ipsam? Expedita quae dolores doloremque dolorum impedit placeat nostrum modi odio."
			},
			{
				title: "Assumenda, sed,",
				body: "Lorem ipsum dolor sit, amet consectetur adipisicing elit. Assumenda, sed, numquam rerum nihil est quis sint temporibus voluptatem, at aliquam labore deleniti facilis fugit suscipit!"
			},
			{
				title: "Nisi unde a nihil",
				body: "Lorem ipsum dolor sit amet consectetur, adipisicing elit. Nisi unde a nihil exercitationem? Impedit nulla consectetur temporibus aspernatur veniam maxime similique libero fugiat, maiores laboriosam?"
			},
			{
				title: "Laborum magnam placeat enim",
				body: "Lorem ipsum, dolor sit amet consectetur adipisicing elit. Laborum magnam placeat enim, ipsa accusantium facilis necessitatibus. Optio doloremque assumenda, facilis necessitatibus ducimus unde ab error?"
			},
			{
				title: "Nulla aspernatur facilis ipsam",
				body: "Lorem, ipsum dolor sit amet consectetur adipisicing elit. Nulla aspernatur facilis ipsam deleniti, itaque doloremque nam iusto consectetur sequi quae ipsum ipsa veritatis cumque molestias."
			},
		]
		for (let i = 0; i < 10; i++) {
			list.unshift({
				id: uuidv4(),
				timestamp: moment().add(i, "hours").format("LLLL"),
				title: defaultList[i].title,
				body: defaultList[i].body
			})
		}
		db.get('posts')
			.assign({ posts: list })
			.write()
		let newList = db.get('posts').value()
		posts = newList
	}
	const save = () => {
		const adapter = new LocalStorage('db')
		const db = low(adapter)
		let list = db.get('posts').value()
		let stringed = JSON.stringify(list)
		let blob = new Blob([stringed], {type: "application/json"})
		FileSaver.saveAs(blob, "test.json")
	}
	const saveText = () => {
		const adapter = new LocalStorage('db')
		const db = low(adapter)
		let list = db.get('posts').value()
		let stringed = list.map(each => {
			let title = each.title.replace(/\r|\n/gi, "")
			let body = each.body.replace(/\r|\n/gi, "")
			let timestamp = each.timestamp
			return `${title} (${timestamp}) - ${body}\r\n\r\n`
		})
		let blob = new Blob(stringed, {type: "text/plain"})
		FileSaver.saveAs(blob, "test.txt")
	}
	const loadFile = async (input) => {
		const adapter = new LocalStorage('db')
		const db = low(adapter)
		const selectedFile = document.getElementById('load-input').files[0]
		let reader = new FileReader()
		reader.onload = (event) => {
			let res = JSON.parse(event.target.result)
			posts = res
			db.set('posts', res)
				.write()
		}
		reader.readAsText(selectedFile)
	}
	onMount(async () => {
		const adapter = new LocalStorage('db')
		const db = low(adapter)
		if (db.has('posts').value()) {
			let existing = db.get('posts').value()
			posts = existing
		} else {
			let lorem = [
				{
					title: "Rerum, velit molestias",
					body: "Lorem ipsum dolor sit, amet consectetur adipisicing elit. Rerum, velit molestias eveniet, animi ipsum praesentium ea voluptatibus necessitatibus eius amet incidunt quod odio vero saepe."
				},
				{
					title: "Laborum itaque maxime saepe",
					body: "Lorem, ipsum dolor sit amet consectetur adipisicing elit. Laborum itaque maxime saepe nihil. Laboriosam, eos neque? Recusandae doloremque praesentium enim vero, asperiores ratione soluta ad."
				},
				{
					title: "Beatae numquam repudiandae recusandae",
					body: "Lorem ipsum dolor sit amet, consectetur adipisicing elit. Beatae numquam repudiandae recusandae libero. Numquam in illo ratione porro unde amet omnis incidunt sed aut quos."
				},
				{
					title: "Nihil provident ipsam,",
					body: "Lorem, ipsum dolor sit amet consectetur adipisicing elit. Nihil provident ipsam, iste, commodi suscipit assumenda dolore sequi architecto veniam totam magni aspernatur est consequuntur eligendi."
				},
				{
					title: "Rerum dolor eaque similique",
					body: "Lorem ipsum dolor sit, amet consectetur adipisicing elit. Rerum dolor eaque similique fugiat? Corrupti et ipsam eligendi quae, ducimus quaerat ea doloremque! Earum, eaque molestias!"
				},
				{
					title: "Possimus illum est ullam",
					body: "Lorem ipsum dolor sit, amet consectetur adipisicing elit. Possimus illum est ullam, dolorum ipsa ipsam? Expedita quae dolores doloremque dolorum impedit placeat nostrum modi odio."
				},
				{
					title: "Assumenda, sed,",
					body: "Lorem ipsum dolor sit, amet consectetur adipisicing elit. Assumenda, sed, numquam rerum nihil est quis sint temporibus voluptatem, at aliquam labore deleniti facilis fugit suscipit!"
				},
				{
					title: "Nisi unde a nihil",
					body: "Lorem ipsum dolor sit amet consectetur, adipisicing elit. Nisi unde a nihil exercitationem? Impedit nulla consectetur temporibus aspernatur veniam maxime similique libero fugiat, maiores laboriosam?"
				},
				{
					title: "Laborum magnam placeat enim",
					body: "Lorem ipsum, dolor sit amet consectetur adipisicing elit. Laborum magnam placeat enim, ipsa accusantium facilis necessitatibus. Optio doloremque assumenda, facilis necessitatibus ducimus unde ab error?"
				},
				{
					title: "Nulla aspernatur facilis ipsam",
					body: "Lorem, ipsum dolor sit amet consectetur adipisicing elit. Nulla aspernatur facilis ipsam deleniti, itaque doloremque nam iusto consectetur sequi quae ipsum ipsa veritatis cumque molestias."
				},
			]
			let list = []
			for (let each of lorem) {
				list.unshift({
					id: uuidv4(),
					timestamp: moment().format("LLLL"),
					title: each.title,
					body: each.body
				})
			}
			db.defaults({ posts: list }).write()
			let newList = db.get('posts').value()
			posts = newList
		}
	})
</script>

<style lang="sass">
#main-wrapper
	position: relative
	padding: 0.25rem
	background: #ddd
	max-width: 90rem
	margin: 2.5rem auto 0.5rem auto
	border-radius: 0.25rem
	#add-post
		display: flex
		flex-wrap: wrap
		align-items: center
		justify-content: center
		align-items: stretch
		margin: 0.5rem
		border-radius: 0.25rem
		overflow: hidden
		background: #eee
		box-shadow: 0 0 0.33rem 0.1rem rgba(black, 0.1)
		#title
			display: flex
			flex-wrap: wrap
			align-items: center
			justify-content: center
			flex: 1
			min-width: 100%
			#title-label
				display: flex
				flex-wrap: wrap
				align-items: center
				justify-content: center
				justify-content: flex-start
				padding: 0.25rem
				width: 100%
				margin-left: 0.25rem
				font-size: 0.9rem
			#title-text
				flex: 1
				margin: 0 0.5rem 0.5rem 0.5rem
				background: rgba(white, 0.5)
				box-shadow: 0 0 0.33rem 0.1rem rgba(black, 0.1) inset
				border: 0
				padding: 0.5rem
				border-radius: 0.25rem
				&::placeholder
		#body
			display: flex
			flex-wrap: wrap
			align-items: center
			justify-content: center
			flex: 1
			min-width: 100%
			#body-label
				display: flex
				flex-wrap: wrap
				align-items: center
				justify-content: center
				justify-content: flex-start
				padding: 0.25rem
				width: 100%
				margin-left: 0.25rem
				font-size: 0.9rem
			#body-text
				flex: 1
				margin: 0 0.5rem 0.5rem 0.5rem
				border: 0
				padding: 0.5rem
				background: rgba(white, 0.5)
				box-shadow: 0 0 0.33rem 0.1rem rgba(black, 0.1) inset
				border-radius: 0.25rem
				font-family: 'Roboto', sans-serif
				&::placeholder
		#enter-post
			width: 100%
			max-width: 20rem
			padding: 0.5rem
			margin: 0.25rem
			cursor: pointer
			display: flex
			flex-wrap: wrap
			// background: #ccc
			align-items: center
			justify-content: center
			border-radius: 0.25rem
			box-shadow: 0 0 0.33rem 0.1rem rgba(black, 0.1)
			transition: all 0.2s ease-in-out
			&:hover
				box-shadow: 0 0 0.33rem 0.1rem rgba(black, 0.2)
		#options
			display: flex
			flex-wrap: wrap
			align-items: center
			justify-content: center
			align-items: stretch
			width: 100%
			margin-top: 0.5rem
			.options-option
				width: auto
				flex-grow: 1
				margin: 0.25rem
				border-radius: 0.25rem
				box-shadow: 0 0 0.33rem 0.1rem rgba(black, 0.1)
			#save
				display: flex
				flex-wrap: wrap
				align-items: center
				justify-content: center
				padding: 0.25rem
				.save-title
					padding: 0.25rem
				.save-text
					padding: 0.25rem 0.5rem
					margin: 0 0.25rem
					cursor: pointer
					background: #bbb
					transition: background 0.2s ease-in-out
					border-radius: 0.25rem
					&:hover
						background: #ccc
			#load
				display: flex
				flex-wrap: wrap
				align-items: center
				justify-content: center
				padding: 0.25rem
				#load-text
					display: flex
					flex-wrap: wrap
					align-items: center
					justify-content: center
					padding: 0.25rem
					margin-right: 0.5rem
				#load-input-wrapper
					display: flex
					flex-wrap: wrap
					align-items: center
					justify-content: center
					text-align: center
				#load-input
					padding: 0.25rem
			#search
				display: flex
				flex-wrap: wrap
				align-items: center
				justify-content: center
				padding: 0.25rem
				#search-title
					margin-right: 0.5rem
					display: flex
					flex-wrap: wrap
					align-items: center
					justify-content: center
				#search-input
					color: black
	#posts-wrapper
		display: flex
		flex-wrap: wrap
		align-items: center
		justify-content: center
		align-items: stretch
		border-radius: 0.25rem
		overflow: hidden
		padding: 0.25rem
</style>

<Navigation />
<div id="main-wrapper">
	<div id="add-post">
		<div id="title">
			<label id="title-label" for="title">Title</label>
			<input id="title-text" type="text" name="title" placeholder="input text here">
		</div>
		<div id="body">
			<label id="body-label" for="body">Body</label>
			<textarea id="body-text" type="text" name="body" rows="5" placeholder="input text here"></textarea>
			<!-- <textarea id="body-text" type="text" name="body" rows="5" placeholder="input text here" @keyup.enter="write"></textarea> -->
		</div>
		<p id="enter-post" on:click={write}>Add</p>
		<div id="options">
			<div id="save" class="options-option">
				<p class="save-title">Save As: </p>
				<p class="save-text" on:click={saveText}>Text</p>
				<p class="save-text" on:click={save}>JSON</p>
			</div>
			<div id="load" class="options-option">
				<p id="load-text">Load (.JSON)</p>
				<div id="load-input-wrapper">
					<input type="file" id="load-input" on:change={loadFile} accept=".json">
				</div>
			</div>
			<div id="search" class="options-option">
				<p id="search-title">Search: </p>
				<input type="text" on:change={search} on:input={search} id="search-input">
			</div>
		</div>
	</div>
	<div id="posts-wrapper">
		{#each posts as post, index}
			<Post info={post} on:postUpdate={updateList} on:postDelete={updateList} />
		{/each}
	</div>
</div>
