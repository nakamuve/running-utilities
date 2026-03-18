<script>
	import '../app.css';
</script>

<div class="flex min-h-screen flex-col bg-gray-100">
	<!-- Header/Navbar -->
	<header class="bg-primary text-white shadow">
		<div class="container mx-auto px-4 py-6">
			<div class="flex items-center justify-between">
				<h1 class="text-3xl font-bold">
					<a href="/">Nakamuve</a>
				</h1>
			</div>
		</div>
	</header>

	<!-- Main Content -->
	<main class="container mx-auto flex-grow px-4 py-8">
		<slot />
	</main>

	<!-- Footer -->
	<footer class="footer footer-center bg-primary p-4 text-white">
		<div>
			<p>Copyright © {new Date().getFullYear()} - Nakamuve</p>
			<div class="flex justify-center gap-3">
				<a href="https://utas.me/barebearfoot-gan" class="hover:text-secondary underline"
					>Contact Us</a
				>
			</div>
		</div>
	</footer>
</div>
