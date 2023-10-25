<script>
    import { onMount } from 'svelte';
    import { slide } from 'svelte/transition';
     let startTime;
     let endTime;
     let gameInProgress = false;
     let colorChanged = false;
     let colorChangeTimeout;
     let gameCount = 0;
     let responseTimes = [];
     let leaderboard = [];
     let studentId = '';
     let name = '';
     let avgTime = '';
     let status = "게임 시작";
   
     onMount(() => {
    const savedLeaderboard = localStorage.getItem('leaderboard');
    if(savedLeaderboard) {
      leaderboard = JSON.parse(savedLeaderboard);
    }
  });
     const resetAll = () => {
    resetGame();
    gameCount = 0;
    responseTimes = [];
    avgTime = '';
    studentId = '';
    name = '';
  }

     const startGame = () => {
        status = "게임 진행중"
    if (gameCount === 5) {
      avgTime = responseTimes.reduce((a, b) => a + b, 0) / responseTimes.length;
      status = "게임 시작"
      alert('게임이 끝났습니다. 학번과 이름을 입력해주세요.'); // 게임이 끝났음을 알리는 메시지 추가
      return;
    }

    gameInProgress = true;
    const randomTime = Math.random() * 2000 + 1000;
    colorChangeTimeout = setTimeout(changeColor, randomTime);
  }
   
     const changeColor = () => {
       colorChanged = true;
       startTime = Date.now();
     }
   
     const endGame = () => {
       if (!colorChanged) {
        status = "게임 시작"
         resetGame();
         alert('아직 색이 바뀌지 않았습니다. 다시 시작해주세요.');
         return;
       }
       endTime = Date.now();
       gameInProgress = false;
       colorChanged = false;
       responseTimes.push(endTime - startTime);
       gameCount++;
       startGame();
     }

     const resetGame = () => {
       clearTimeout(colorChangeTimeout);
       gameCount = 0;
       responseTimes = [];
       colorChanged = false;
       gameInProgress = false;
     }
   
     const submitScore = () => {
    const existingEntryIndex = leaderboard.findIndex(entry => entry.studentId === studentId);

    if (existingEntryIndex !== -1) {
      leaderboard[existingEntryIndex] = { studentId, name, avgTime };
    } else {
      leaderboard.push({ studentId, name, avgTime });
    }

    leaderboard = leaderboard.sort((a, b) => a.avgTime - b.avgTime);
    localStorage.setItem('leaderboard', JSON.stringify(leaderboard)); // 리더보드 정보를 로컬 스토리지에 저장
    resetAll()
  }

   </script>
   <div class="w-full h-10 bg-black text-white font-bold text-3xl text-center">반응속도 측정</div>
   <div class="flex justify-between">

    <div>
<div class="flex justify-center">
    <div></div>
<div class="w-full">
   <div class="w-[300px] h-full" id="gameDiv" on:click={endGame} style="background-color: {colorChanged ? 'blue' : 'red'};">
</div>
<div class="bg-blue-500">
<button class="p-3" on:click={startGame}>{status}</button>
</div>
</div>

   {#if gameCount === 0}
   <div class="justify-center">
     <!-- 게임이 5번 진행된 후에만 학번과 이름을 입력할 수 있도록 수정 -->
     <div class="flex justify-between">
   <label for="studentId">학번:</label>
   <input type="text" id="studentId" bind:value={studentId} required>
</div>
   <label for="name">이름:</label>
   <input type="text" id="name" bind:value={name} required>
   <button class="" on:click={submitScore}>제출</button>
   <button on:click={resetAll}>초기화</button>

</div>
 {/if}
</div>
</div>


   <div class="bg-blue-500 text-white m-3 p-3 rounded-3xl text-sm font-bold" id="leaderboard">
    <h1 class="text-center text-xl mb-1">순위표</h1>
     {#each leaderboard as entry, index (entry.studentId)}
       <div in:slide={{ duration: 500 }}>{index + 1}위 {entry.studentId} {entry.name} {entry.avgTime}ms</div>
       {/each}
   </div>


</div>