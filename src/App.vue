<template>
	<div class="container mt-5">
		<h1 class="text-center mb-4">Моя коллекция книг</h1>
		<div class="row justify-content-center mb-4">
			<div class="col-md-8">
				<form @submit.prevent="addBook" class="border p-3 rounded bg-light">
					<div class="mb-3">
						<label class="form-label">Название</label>
						<input
							v-model="newBook.title"
							type="text"
							class="form-control"
							required
						/>
					</div>
					<div class="mb-3">
						<label class="form-label">Описание</label>
						<textarea
							v-model="newBook.description"
							class="form-control"
							rows="2"
							required
						></textarea>
					</div>
					<div class="mb-3">
						<label class="form-label">Изображение (PNG)</label>
						<input
							type="file"
							class="form-control"
							accept="image/png"
							@change="handleImageUpload"
						/>
						<small class="text-muted">Разрешены только файлы PNG</small>
					</div>
					<button type="submit" class="btn btn-primary w-100">Добавить книгу</button>
				</form>
			</div>
		</div>
		<div class="row mb-4">
			<div class="col text-center">
				<button class="btn btn-outline-secondary me-2" @click="sortByTitleAsc">
					Сортировать по названию (А→Я)
				</button>
				<button class="btn btn-outline-secondary me-2" @click="sortByTitleDesc">
					Сортировать по названию (Я→А)
				</button>
				<button class="btn btn-outline-secondary" @click="sortByDate">
					Сортировать по дате (сначала новые)
				</button>
			</div>
		</div>
		<div class="row">
			<div
				v-for="book in sortedBooks"
				:key="book.id"
				class="col-md-4 mb-4"
			>
				<div class="card h-100">
					<div v-if="book.imageData" class="card-img-top" style="height: 200px; overflow: hidden;">
						<img :src="book.imageData" alt="Обложка книги" 
							style="width: 100%; height: 100%; object-fit: cover;">
					</div>
					<div v-else class="card-img-top d-flex align-items-center justify-content-center bg-secondary" 
							style="height: 200px;">
						<span class="display-1 text-white opacity-75">📖</span>
					</div>
					<div class="card-body">
						<h5 class="card-title">{{ book.title }}</h5>
						<p class="card-text">{{ book.description }}</p>
						<button
							@click="removeBook(book.id)"
							class="btn btn-sm btn-danger"
						>
							Удалить
						</button>
					</div>
					<div class="card-footer text-muted">
						Добавлено: {{ formatDate(book.dateAdded) }}
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script setup>

import { ref, computed } from 'vue'
const books = ref([])
const newBook = ref({
	title: '',
	description: '',
	imageData: null
})

const handleImageUpload = (event) => {
	const file = event.target.files[0]
	if (!file) return

	if (file.type !== 'image/png') {
		alert('Пожалуйста, выберите PNG-изображение.')
		event.target.value = ''
		return
	}

	const reader = new FileReader()
	reader.onload = (e) => {
		newBook.value.imageData = e.target.result
	}

	reader.readAsDataURL(file)
}

const addBook = () => {
	if (!newBook.value.title || !newBook.value.description) return

	books.value.push({
		id: Date.now(),
		title: newBook.value.title,
		description: newBook.value.description,
		imageData: newBook.value.imageData,
		dateAdded: new Date().toISOString()
	})
	newBook.value = {
		title: '',
		description: '',
		imageData: null
	}

	const fileInput = document.querySelector('input[type="file"]')
	if (fileInput) fileInput.value = ''
}

const removeBook = (id) => {
	books.value = books.value.filter(book => book.id !== id)
}

const sortType = ref('date')

const sortByTitleAsc = () => { sortType.value = 'titleAsc' }
const sortByTitleDesc = () => { sortType.value = 'titleDesc' }
const sortByDate = () => { sortType.value = 'date' }

const sortedBooks = computed(() => {
	const list = [...books.value]
	if (sortType.value === 'titleAsc') {
		return list.sort((a, b) => a.title.localeCompare(b.title))
	} else if (sortType.value === 'titleDesc') {
		return list.sort((a, b) => b.title.localeCompare(a.title))
	} else {
		return list.sort((a, b) => new Date(b.dateAdded) - new Date(a.dateAdded))
	}
})

const formatDate = (isoString) => {
	const date = new Date(isoString)
	return date.toLocaleDateString('ru-RU')
}

</script>

