<div class="p-10 bg-gray-100 min-h-screen">
  <!-- Summary -->
  <div class="bg-white p-6 rounded-lg shadow mb-8">
    <h1 class="text-3xl font-bold">{{ quizResult?.testName }}</h1>
    <p class="text-lg"><strong>Score:</strong> {{ quizResult?.score }} / {{ quizResult?.totalQuestions }}</p>
    <p class="text-lg"><strong>Time Taken:</strong> {{ quizResult?.timeTaken }}</p>
  </div>

  <!-- Chart -->
  <div class="flex justify-center mb-8">
    <canvas id="quizPieChart" class="w-64 h-64"></canvas>
  </div>

  <!-- Questions -->
  <div class="space-y-6">
    <h2 class="text-2xl font-bold">Review Questions</h2>
    <div *ngFor="let question of quizResult?.questions" class="bg-white p-6 rounded-lg shadow">
      <p class="font-bold">{{ question.questionText }}</p>
      <ul class="list-disc pl-6">
        <li *ngFor="let option of question.options" 
            [class.text-green-500]="option === question.correctAnswer"
            [class.text-red-500]="option === question.userAnswer && option !== question.correctAnswer">
          {{ option }}
        </li>
      </ul>
      <p *ngIf="question.userAnswer === question.correctAnswer" class="text-green-500 font-bold">✔ Correct</p>
      <p *ngIf="question.userAnswer !== question.correctAnswer" class="text-red-500 font-bold">✖ Incorrect</p>
    </div>
  </div>
</div>
