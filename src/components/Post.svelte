<script>
	import low from "lowdb"
	import moment from "moment"
	import copy from "copy-text-to-clipboard"
	import LocalStorage from "lowdb/adapters/LocalStorage"
	import { createEventDispatcher } from "svelte"
	export let info = {}
	export let link = false
	const dispatch = createEventDispatcher()
	let time = `${moment().format("LL")} ${moment().format("LTS")}`
	let timeShort = moment().format("LTS")
	let title = info.title
	let body = info.body
	const updateTitle = (e) => {
		let newTitle = e.target.textContent
		title = newTitle
	}
	const updateBody = (e) => {
		let newBody = e.target.textContent
		body = newBody
	}
	const deletePost = () => {
		const adapter = new LocalStorage('db')
		const db = low(adapter)
		db.get('posts')
			.remove({ id: info.id })
			.write()
		dispatch('postDelete', {id: info.id})
	}
	const copyNote = () => {
		copy(`${info.title} (${info.timestamp}): ${info.body}`)
	}
	const updatePost = () => {
		e.target.blur()
		const adapter = new LocalStorage('db')
		const db = low(adapter)
		db.get('posts')
			.find({ id: info.id })
			.assign({ title: title, body: body })
			.write()
		dispatch('postUpdate', {id: info.id})
	}
</script>

<style lang="sass">
.post
	display: flex
	flex-wrap: wrap
	align-items: center
	justify-content: center
	position: relative
	animation: fadeIn 0.3s ease-in-out
	flex: 1
	min-width: 100%
	overflow: hidden
	margin: 0.5rem
	border-radius: 0.25rem
	display: flex
	flex-wrap: wrap
	align-items: center
	justify-content: center
	flex-direction: column
	background: #eee
	transition: all 0.2s ease-in-out
	box-shadow: 0 0 0.5rem 0.1rem rgba(0, 0, 0, 0.1)
	@media (min-width: 42rem)
		min-width: 40%
		max-width: 55%
	&:hover
		box-shadow: 0 0 0.5rem 0.1rem rgba(0, 0, 0, 0.15)
	.post-buttons
		display: flex
		flex-wrap: wrap
		align-items: center
		justify-content: center
		position: absolute
		bottom: 0
		left: 0
		padding: 0.25rem
		.post-button
			width: 1.75rem
			height: 1.75rem
			display: flex
			flex-wrap: wrap
			align-items: center
			justify-content: center
			transition: background 0.2s ease-in-out
			&:hover
				.post-button-image
					transform: scale(1.25)
			.post-button-image
				height: 1.25rem
				width: 1.25rem
				box-sizing: content-box
				transition: all 0.1s ease-in-out
		#save
			cursor: pointer
			margin: 0.1rem
			padding: 0.25rem
			border-radius: 0.25rem
			background: rgba(green, 0.25)
			&:hover
				background: rgba(green, 0.5)
		#delete
			margin: 0.1rem
			padding: 0.25rem
			cursor: pointer
			border-radius: 0.25rem
			background: rgba(red, 0.25)
			&:hover
				background: rgba(red, 0.5)
		#copy
			margin: 0.1rem
			padding: 0.25rem
			cursor: pointer
			border-radius: 0.25rem
			background: rgba(blue, 0.25)
			&:hover
				background: rgba(blue, 0.5)
	.post-title
		display: flex
		flex-wrap: wrap
		align-items: center
		justify-content: center
		min-width: 100%
		padding: 1rem
		font-size: 1.25rem
		text-decoration: none
		color: rgba(black, 0.9)
		&.linked
			cursor: pointer
	.post-body
		display: flex
		flex-wrap: wrap
		align-items: center
		justify-content: center
		min-width: 100%
		padding: 1.5rem
		flex: auto
		text-indent: 0.5rem
		word-break: break-word
	.post-time
		display: flex
		flex-wrap: wrap
		align-items: center
		justify-content: center
		padding: 0.25rem
		width: 100%
		justify-content: flex-end
		font-size: 0.75rem
		color: rgba(black, 0.9)
</style>

<div class="post" data-id={info.id}>
	{#if link}
		<a class="{ link ? 'post-title linked' : 'post-title' }" href={`/${info.id}`}>{info.title}</a>
	{/if}
	{#if !link}
		<p class="post-title" contenteditable="true" on:input={updateTitle}>{info.title}</p>
	{/if}
	<!-- <p class="post-title" contenteditable="true" on:input={updateTitle} @keyup.enter="updatePost" v-if="!this.link">{info.title}</p> -->
	<p class="post-body" contenteditable="true" on:input={updateBody}>{info.body}</p>
	<!-- <p class="post-body" contenteditable="true" on:input={updateBody} @keyup.enter="updatePost">{info.body}</p> -->
	<p class="post-time">{info.timestamp}</p>
	<div class="post-buttons">
		<div class="post-button" id="delete" on:click={deletePost}>
			<img class="post-button-image" src="/icons/x.svg" alt="delete" />
		</div>
		<div class="post-button" id="save" on:click={updatePost}>
			<img class="post-button-image" src="/icons/save.svg" alt="save" />
		</div>
		<div class="post-button" id="copy" on:click={copyNote}>
			<img class="post-button-image" src="/icons/copy.svg" alt="copy" />
		</div>
	</div>
</div>
