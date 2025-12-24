const { createCanvas } = require('canvas');
const fs = require('fs');

// --- Configuration ---
const CANVAS_SIZE = 400;
const NUM_STREAKS = 150;
const NOISE_DOTS = 1000;
const NOISE_PIXELS = 4000;

// --- Setup ---
const staticCanvas = createCanvas(CANVAS_SIZE, CANVAS_SIZE);
const ctx_static = staticCanvas.getContext('2d');
const streaksData = []; // To store parameters for each streak

// Helper function to save the canvas to a file
function saveCanvas(filename) {
    return new Promise((resolve, reject) => {
        const out = fs.createWriteStream(__dirname + '/' + filename);
        const stream = staticCanvas.createPNGStream();
        stream.pipe(out);
        out.on('finish', () => {
            console.log(`Saved: ${filename}`);
            resolve();
        });
        out.on('error', reject);
    });
}

// --- Stage 1: Generate Base Image (Streaks) ---
function generateBaseImage(ctx) {
    // Clear the canvas to black
    ctx.fillStyle = 'black';
    ctx.fillRect(0, 0, CANVAS_SIZE, CANVAS_SIZE);
    
    // Add initial low-intensity background noise (from previous iteration)
    ctx.fillStyle = 'rgba(255, 255, 255, 0.02)'; // Very low opacity white
    for (let i = 0; i < NOISE_DOTS; i++) {
        const x = Math.random() * CANVAS_SIZE;
        const y = Math.random() * CANVAS_SIZE;
        ctx.fillRect(x, y, 1, 1); // Draw a 1x1 pixel dot
    }

    // Draw streaks and store their data
    for (let i = 0; i < NUM_STREAKS; i++) {
        const x = Math.round(Math.random() * CANVAS_SIZE);
        const y = Math.round(Math.random() * CANVAS_SIZE * 0.9);
        const length = 10 + Math.random() * 40;
        const alpha = 0.1 + Math.random() * 0.7; // Wider random alpha range
        const lineWidth = 1 + Math.random();

        // Store data for later use (vector drawing)
        streaksData.push({ x, y, length, alpha, lineWidth });

        // Create a vertical linear gradient for the streak
        const streakGrad = ctx.createLinearGradient(x, y, x, y + length);
        // Fades from bright white to transparent (High intensity at y, low at y+length)
        streakGrad.addColorStop(0, `rgba(255, 255, 255, ${alpha})`);
        streakGrad.addColorStop(0.8, `rgba(255, 255, 255, ${alpha * 0.5})`);
        streakGrad.addColorStop(1, 'rgba(0, 0, 0, 0)');

        ctx.strokeStyle = streakGrad;
        ctx.lineWidth = lineWidth;

        // Draw the vertical line (streak)
        ctx.beginPath();
        ctx.moveTo(x, y);
        ctx.lineTo(x, y + length);
        ctx.stroke();
    }
}

// --- Stage 2: Add Comprehensive Synthetic Noise Simulation ---
function addComprehensiveNoise(ctx) {
    // Save current state before applying global effects
    ctx.save();

    // 1. Uniform Noise (Very faint gray layer)
    ctx.fillStyle = 'rgba(100, 100, 100, 0.005)';
    ctx.fillRect(0, 0, CANVAS_SIZE, CANVAS_SIZE);

    // 2. Periodic Noise (Faint sinusoidal stripes)
    ctx.strokeStyle = 'rgba(255, 255, 255, 0.01)';
    ctx.lineWidth = 0.5;
    for (let i = 0; i < CANVAS_SIZE; i += 10) {
        ctx.beginPath();
        ctx.moveTo(i, 0);
        ctx.lineTo(i, CANVAS_SIZE);
        ctx.stroke();
    }

    // 3. Speckle Noise (Random multiplicative pattern)
    ctx.fillStyle = 'rgba(255, 255, 255, 0.008)';
    for (let i = 0; i < 500; i++) {
        const x = Math.random() * CANVAS_SIZE;
        const y = Math.random() * CANVAS_SIZE;
        const size = 5 + Math.random() * 10;
        ctx.fillRect(x, y, size, size);
    }

    // 4. Gaussian Noise (Simulated with many tiny, low-opacity, random dots)
    ctx.fillStyle = 'rgba(255, 255, 255, 0.05)';
    for (let i = 0; i < 2000; i++) {
        const x = Math.random() * CANVAS_SIZE;
        const y = Math.random() * CANVAS_SIZE;
        ctx.fillRect(x, y, 1, 1);
    }

    // 5. Salt-and-Pepper Noise (Impulse Noise)
    for (let i = 0; i < NOISE_PIXELS; i++) {
        const x = Math.floor(Math.random() * CANVAS_SIZE);
        const y = Math.floor(Math.random() * CANVAS_SIZE);
        const color = Math.random() < 0.5 ? 'rgba(255, 255, 255, 0.9)' : 'rgba(0, 0, 0, 0.9)';
        ctx.fillStyle = color;
        ctx.fillRect(x, y, 1, 1);
    }

    // 6. Compression Artifacts (Blockiness)
    const blockSize = 16;
    ctx.fillStyle = 'rgba(255, 255, 255, 0.003)';
    for (let x = 0; x < CANVAS_SIZE; x += blockSize) {
        for (let y = 0; y < CANVAS_SIZE; y += blockSize) {
            if (Math.random() < 0.2) {
                ctx.fillRect(x, y, blockSize, blockSize);
            }
        }
    }

    // 7. Motion Blur (Simulated by a slight, faint shift)
    ctx.globalAlpha = 0.01;
    ctx.drawImage(staticCanvas, 2, 0);
    ctx.globalAlpha = 1.0;

    // 8. Illumination/Glare (from previous comprehensive noise)
    const glareGrad = ctx.createRadialGradient(
        CANVAS_SIZE * 0.7, CANVAS_SIZE * 0.3, 0,
        CANVAS_SIZE * 0.7, CANVAS_SIZE * 0.3, CANVAS_SIZE * 0.5
    );
    glareGrad.addColorStop(0, 'rgba(255, 255, 255, 0.05)');
    glareGrad.addColorStop(1, 'rgba(0, 0, 0, 0)');
    ctx.fillStyle = glareGrad;
    ctx.fillRect(0, 0, CANVAS_SIZE, CANVAS_SIZE);

    // Restore state
    ctx.restore();
}

// --- Stage 4: Draw Velocity Vectors ---
function drawVelocityVectors(ctx, data) {
    // The streaks are vertical lines, so "least square curve fitting" is trivial.
    // We determine the flow direction based on the intensity gradient:
    // High Intensity (start of streak) -> Low Intensity (end of streak)
    
    const ARROW_COLOR = 'rgba(255, 0, 0, 0.9)'; // Red for visibility
    const ARROW_SIZE = 5; // Size of the arrowhead

    ctx.strokeStyle = ARROW_COLOR;
    ctx.fillStyle = ARROW_COLOR;
    ctx.lineWidth = 1.5;

    data.forEach(streak => {
        const { x, y, length } = streak;
        
        // Vector starts at (x, y) (high intensity) and ends at (x, y + length) (low intensity)
        const startX = x;
        const startY = y;
        const endX = x;
        const endY = y + length;
        
        // Draw the arrowhead at the end point (y + length)
        // Since the vector is vertical (downwards), the arrowhead points down.
        ctx.beginPath();
        ctx.moveTo(endX, endY);
        ctx.lineTo(endX - ARROW_SIZE / 2, endY - ARROW_SIZE);
        ctx.lineTo(endX + ARROW_SIZE / 2, endY - ARROW_SIZE);
        ctx.closePath();
        ctx.fill();
    });
}

// --- Main Execution Flow ---
async function run() {
    // --- Stage 1: Generate Base Image (Streaks) ---
    generateBaseImage(ctx_static);
    await saveCanvas('stage1_base_streaks.png');

    // --- Stage 2: Add Noise ---
    // The canvas is now covered with the base streaks. We apply the noise on top.
    addComprehensiveNoise(ctx_static);
    await saveCanvas('stage2_noisy_image.png');

    // --- Stage 3: Remove Noise (Conceptual Denoising) ---
    // We re-generate the clean image to simulate a successful denoising step
    // and prepare a clean canvas for the vector analysis.
    // NOTE: We must clear the streaksData array before calling generateBaseImage again
    // to prevent duplicate data, then re-populate it.
    streaksData.length = 0; 
    generateBaseImage(ctx_static);
    await saveCanvas('stage3_denoised_image.png');

    // --- Stage 4: Add Velocity Vectors ---
    // The vector analysis is applied to the clean streak data.
    drawVelocityVectors(ctx_static, streaksData);
    await saveCanvas('stage4_final_vectors.png');

    // Draw the crosshair on the final image (from original code)
    const s1x = Math.round(CANVAS_SIZE * Math.random());
    const s1y = Math.round(CANVAS_SIZE * Math.random());
    const s1Grad = ctx_static.createRadialGradient(s1x, s1y, 0, s1x, s1y, 20);
    s1Grad.addColorStop(0, 'rgba(255, 255, 255, 1)');
    s1Grad.addColorStop(1, 'rgba(255, 255, 255, 0)');
    ctx_static.strokeStyle = s1Grad;
    ctx_static.lineWidth = 1;

    ctx_static.beginPath();
    ctx_static.moveTo(s1x - 4, s1y);
    ctx_static.lineTo(s1x + 4, s1y);
    ctx_static.stroke();

    ctx_static.beginPath();
    ctx_static.moveTo(s1x, s1y - 4);
    ctx_static.lineTo(s1x, s1y + 4);
    ctx_static.stroke();
}

run();

