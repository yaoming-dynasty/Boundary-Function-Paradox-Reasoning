const epsilonInput = document.querySelector('#epsilonRange');
const depthInput = document.querySelector('#depthRange');
const epsilonValue = document.querySelector('#epsilonValue');
const depthValue = document.querySelector('#depthValue');
const reciprocalValue = document.querySelector('#reciprocalValue');
const reachValue = document.querySelector('#reachValue');
const archetype = document.querySelector('#archetype');
const summary = document.querySelector('#summary');
const canvas = document.querySelector('#graphCanvas');
const ctx = canvas.getContext('2d');

function clamp(value, min, max) {
  return Math.min(Math.max(value, min), max);
}

function formatNumeric(value) {
  if (value >= 1000) return value.toFixed(0);
  if (value >= 10) return value.toFixed(2);
  if (value >= 1) return value.toFixed(3);
  return value.toFixed(3);
}

function getReachLabel(reciprocal) {
  if (reciprocal >= 100) return 'Infinite-scale';
  if (reciprocal >= 25) return 'Expansive';
  if (reciprocal >= 10) return 'Strong';
  if (reciprocal >= 4) return 'Moderate';
  if (reciprocal >= 1.5) return 'Emergent';
  return 'Localized';
}

function buildArchetype(depth) {
  const layers = [
    '0',
    '(0)',
    '((0))',
    '((0)))',
    '((0))) ~',
    '((0))) ~ INFINITY'
  ];

  const visibleDepth = clamp(depth, 0, 5);
  let sequence = '';

  if (visibleDepth === 0) {
    sequence = '0';
  } else if (visibleDepth === 1) {
    sequence = '(0)';
  } else if (visibleDepth === 2) {
    sequence = '((0))';
  } else if (visibleDepth === 3) {
    sequence = '((0)))';
  } else if (visibleDepth === 4) {
    sequence = '((0))) ~';
  } else {
    sequence = '((0))) ~ INFINITY';
  }

  return sequence;
}

function updateSummary(x, reciprocal, depth) {
  const statement = `With x = ${formatNumeric(x)}, the reciprocal is ${formatNumeric(reciprocal)}. As the local margin shrinks, the system scales outward.`;

  const stage = depth >= 4
    ? 'The directional operator ~ has launched the boundary beyond local closure.'
    : depth >= 2
      ? 'The system is still mapping its adjacent dependencies.'
      : 'The seed remains anchored in its localized origin.';

  summary.textContent = `${statement} ${stage}`;
}

function drawGraph() {
  const width = canvas.width;
  const height = canvas.height;
  const padding = 28;

  ctx.clearRect(0, 0, width, height);

  ctx.strokeStyle = '#2a3848';
  ctx.lineWidth = 1;

  // axes
  ctx.beginPath();
  ctx.moveTo(padding, padding);
  ctx.lineTo(padding, height - padding);
  ctx.lineTo(width - padding, height - padding);
  ctx.stroke();

  // gridlines
  for (let i = 0; i <= 5; i += 1) {
    const y = padding + (i / 5) * (height - padding * 2);
    ctx.beginPath();
    ctx.moveTo(padding, y);
    ctx.lineTo(width - padding, y);
    ctx.strokeStyle = '#1d2a38';
    ctx.stroke();
  }

  // function curve: y = 1 / x from 0.1 to 1.0
  ctx.beginPath();
  ctx.strokeStyle = '#7ee3ff';
  ctx.lineWidth = 2.5;

  for (let px = padding; px <= width - padding; px += 1) {
    const x = 0.1 + ((px - padding) / (width - padding * 2)) * 0.9;
    const y = 1 / x;
    const graphY = height - padding - ((y - 1) / 50) * (height - padding * 2);

    if (px === padding) {
      ctx.moveTo(px, graphY);
    } else {
      ctx.lineTo(px, graphY);
    }
  }

  ctx.stroke();

  // highlight current x
  const xValue = Number(epsilonInput.value);
  const reciprocal = 1 / xValue;
  const highlightX = padding + ((xValue - 0.1) / 0.9) * (width - padding * 2);
  const highlightY = height - padding - ((reciprocal - 1) / 50) * (height - padding * 2);

  ctx.fillStyle = '#ffb86c';
  ctx.beginPath();
  ctx.arc(highlightX, highlightY, 5, 0, Math.PI * 2);
  ctx.fill();

  ctx.fillStyle = '#8b9bb0';
  ctx.font = '12px sans-serif';
  ctx.fillText('x → 0', padding + 4, height - 8);
  ctx.fillText('1/x', width - 40, padding + 12);
}

function update() {
  const xValue = Number(epsilonInput.value);
  const depthValueNumber = Number(depthInput.value);
  const reciprocal = 1 / xValue;

  epsilonValue.textContent = xValue.toFixed(2);
  depthValue.textContent = String(depthValueNumber);
  reciprocalValue.textContent = formatNumeric(reciprocal);
  reachValue.textContent = getReachLabel(reciprocal);

  archetype.textContent = buildArchetype(depthValueNumber);
  updateSummary(xValue, reciprocal, depthValueNumber);
  drawGraph();
}

epsilonInput.addEventListener('input', update);
depthInput.addEventListener('input', update);

update();
