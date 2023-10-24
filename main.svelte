<script>
    import { onMount } from 'svelte';
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
   
   <button on:click={startGame}>{status}</button>
   <div id="gameDiv" on:click={endGame} style="width: 100px; height: 100px; background-color: {colorChanged ? 'red' : 'white'};"></div>
   
   {#if gameCount === 5} <!-- 게임이 5번 진행된 후에만 학번과 이름을 입력할 수 있도록 수정 -->
   <label for="studentId">학번:</label>
   <input type="text" id="studentId" bind:value={studentId} required>
   <label for="name">이름:</label>
   <input type="text" id="name" bind:value={name} required>
   <button on:click={submitScore}>제출</button>
   <button on:click={resetAll}>초기화</button>
 {/if}
   
   <div id="leaderboard">
     {#each leaderboard as entry, index (entry.studentId)}
       <p>{index + 1}. {entry.name} ({entry.studentId}): {entry.avgTime}ms</p>
     {/each}
   </div>
