<template>
	<main class="app">
		<h1>The Quiz</h1>

		<section v-if="loading">
			<p>Loading questions...</p>
		</section>

		<section class="quiz" v-else-if="!quizCompleted && questions.length > 0">
			<div class="quiz-info">
				<span class="question">{{ getCurrentQuestion.question }}</span>
				<span class="score">Score {{ score }}/{{ questions.length }}</span>
			</div>

			<!-- Replacing the options section -->
			<div class="options">
				<label v-for="(option, index) in getCurrentQuestion.options" :for="'option' + index" :class="`option ${getCurrentQuestion.selected == index
					? index == getCurrentQuestion.correct_answer
						? 'correct'
						: 'wrong'
					: ''
					} ${getCurrentQuestion.selected != null && index != getCurrentQuestion.selected
						? 'disabled'
						: ''
					}`">
					<input type="radio" :id="'option' + index" :name="getCurrentQuestion.index" :value="index"
						v-model="getCurrentQuestion.selected" :disabled="getCurrentQuestion.selected" @change="SetAnswer" />
					<span>{{ option }}</span>
				</label>
			</div>

			<button @click="NextQuestion" :disabled="!getCurrentQuestion.selected">
				{{
					getCurrentQuestion.index === questions.length - 1
					? 'Finish'
					: getCurrentQuestion.selected === null
						? 'Select an option'
						: 'Next question'
				}}
			</button>
		</section>

		<section v-else>
			<h2>You have finished the quiz!</h2>
			<p>Your score is {{ score }}/{{ questions.length }}</p>
		</section>
	</main>
</template>
  
<script setup>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';

const loading = ref(true);
const questions = ref([]);
const quizCompleted = ref(false);
const currentQuestion = ref(0);

const fetchData = async () => {
	try {
		const response = await axios.get('https://opentdb.com/api.php?amount=10');
		const data = response.data;
		questions.value = data.results.map((question, index) => {
			const options = [...question.incorrect_answers, question.correct_answer].sort(() => Math.random() - 0.5);
			return {
				...question,
				options,
				selected: null,
				correct_answer: options.indexOf(question.correct_answer), // Assign the correct index
				index,
			};
		});
		loading.value = false;
	} catch (error) {
		console.error('Error fetching questions:', error);
		loading.value = false;
	}
};

const getOptionClass = (optionIndex) => {
	const isSelected = currentQuestion.value !== null && currentQuestion.value.selected === optionIndex;
	const isCorrect = isSelected && optionIndex === currentQuestion.value.correct_answer;
	const isWrong = isSelected && optionIndex !== currentQuestion.value.correct_answer;
	const isDisabled = isSelected;

	return {
		option: true,
		correct: isCorrect,
		wrong: isWrong,
		disabled: isDisabled,
	};
};

onMounted(() => {
	fetchData();
});

const score = computed(() => {
	let value = 0;
	questions.value.forEach((q) => {
		// Using String comparison to handle type mismatch between correct_answer and selected
		if (q.selected !== null && String(q.correct_answer) === String(q.selected)) {
			value++;
		}
	});
	return value;
});

const getCurrentQuestion = computed(() => {
	return questions.value[currentQuestion.value];
});

const SetAnswer = (e) => {
	questions.value[currentQuestion.value].selected = e.target.value;
};

const NextQuestion = () => {
	if (currentQuestion.value < questions.value.length - 1) {
		currentQuestion.value++;
		return;
	}

	quizCompleted.value = true;
};
</script>
  
  
<style src="./style.css"></style>