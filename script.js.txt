const symbols = ['🍒','🍋','🍊','🍉','⭐','7'];
const reels = [
  document.getElementById('reel1'),
  document.getElementById('reel2'),
  document.getElementById('reel3')
];
const btn = document.getElementById('spin-btn');
const resultEl = document.getElementById('result');
const scoreEl = document.getElementById('score-total');

// Cargar puntuación desde sessionStorage
let score = parseInt(sessionStorage.getItem('score')) || 0;
scoreEl.textContent = 'Puntuación: $' + score;

function spinReel(reel, delay) {
  return new Promise(resolve => {
    const start = Date.now();
    const interval = setInterval(() => {
      const sym = symbols[Math.floor(Math.random() * symbols.length)];
      reel.textContent = sym;
      if (Date.now() - start > delay) {
        clearInterval(interval);
        resolve(sym);
      }
    }, 100);
  });
}

function actualizarPuntuacion(results) {
  const count7 = results.filter(r => r === '7').length;
  const counts = {};
  results.forEach(r => counts[r] = (counts[r] || 0) + 1);
  const hasDoubleOther = Object.values(counts).some(c => c === 2 && !counts['7']);
  const hasTripleOther = Object.values(counts).some(c => c === 3 && !counts['7']);
  let gain = 0;

  if (count7 === 1) gain = 10;
  else if (count7 === 2) gain = 30;
  else if (count7 === 3) gain = 70;
  else if (hasDoubleOther) gain = 25;
  else if (hasTripleOther) gain = 50;

  score += gain;
  sessionStorage.setItem('score', score);
  scoreEl.textContent = 'Puntuación: $' + score;
  return gain;
}

btn.addEventListener('click', async () => {
  btn.disabled = true;
  resultEl.textContent = '';
  const results = await Promise.all([
    spinReel(reels[0], 1000),
    spinReel(reels[1], 1400),
    spinReel(reels[2], 1800)
  ]);
  const gain = actualizarPuntuacion(results);
  if (gain > 0) {
    resultEl.textContent = `¡Has ganado $${gain}! (${results.join(' | ')})`;
    resultEl.style.color = '#ffd700';
  } else {
    resultEl.textContent = `No hay premio. (${results.join(' | ')})`;
    resultEl.style.color = '#ff4d4d';
  }
  btn.disabled = false;
});