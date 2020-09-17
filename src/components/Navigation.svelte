<script>
import moment from 'moment'
import { onMount } from "svelte"
let scroll_old = 0
let scroll_new = 0
let time = `${moment().format("LL")} ${moment().format("LTS")}`
const handleScroll = () => {
	let y = window.scrollY
	scroll_new = y
	let nav_wrapper = document.getElementById("navigation-wrapper")
	if (scroll_new > scroll_old) {
		nav_wrapper.style.top = "-3rem"
		scroll_old = y
	} else if (scroll_new < scroll_old) {
		nav_wrapper.style.top = "0rem"
		scroll_old = y
	}
}
onMount(async () => {
	setInterval(() => {
		time = `${moment().format("LL")} ${moment().format("LTS")}`
	}, 1000)
})
</script>

<style lang="sass">
#navigation-wrapper
	display: flex
	align-items: center
	justify-content: space-between
	background: #eee
	border-radius: 0 0 0.25rem 0.25rem
	position: fixed
	top: 0
	width: 100%
	z-index: 5
	margin: 0 auto
	transition: top 0.15s ease-in-out
	#time
		padding: 0.5rem
		flex: 1
		text-decoration: none
		display: flex
		align-items: center
		justify-content: center
</style>

<svelte:window on:scroll={handleScroll} />

<div id="navigation-wrapper">
	<p id="time">{time}</p>
</div>
