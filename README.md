<!DOCTYPE html>
<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>পোস্টার জেনারেটর</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Hind+Siliguri:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Hind Siliguri', sans-serif;
        }
    </style>
</head>
<body class="bg-gray-100 dark:bg-gray-900 text-gray-800 dark:text-gray-200 flex items-center justify-center min-h-screen p-4">

    <div class="container mx-auto grid grid-cols-1 lg:grid-cols-3 gap-8">
        <!-- Controls Section -->
        <div class="lg:col-span-1 bg-white dark:bg-gray-800 p-6 rounded-2xl shadow-lg h-fit">
            <h2 class="text-2xl font-bold mb-6 border-b pb-3 border-gray-200 dark:border-gray-700 text-indigo-600 dark:text-indigo-400">আপনার তথ্য দিন</h2>
            
            <div class="space-y-6">
                <!-- Location Input -->
                <div>
                    <label for="locationInput" class="block text-lg font-medium mb-2">স্থানের নাম</label>
                    <input type="text" id="locationInput" value="বরিশালকে" placeholder="এখানে স্থানের নাম লিখুন" class="w-full px-4 py-2 bg-gray-50 dark:bg-gray-700 border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-indigo-500 transition">
                </div>

                <!-- Image Upload -->
                <div>
                    <label for="imageUpload" class="block text-lg font-medium mb-2">আপনার ছবি</label>
                    <input type="file" id="imageUpload" accept="image/*" class="w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-lg file:border-0 file:font-semibold file:bg-indigo-50 dark:file:bg-indigo-900 file:text-indigo-700 dark:file:text-indigo-300 hover:file:bg-indigo-100 cursor-pointer">
                </div>

                <!-- Image Controls -->
                <div id="image-controls" class="hidden space-y-4 pt-4 border-t border-gray-200 dark:border-gray-700">
                    <div>
                        <label for="zoom" class="block text-sm font-medium">জুম</label>
                        <input type="range" id="zoom" min="0.1" max="3" step="0.05" value="1" class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer dark:bg-gray-700">
                    </div>
                     <div>
                        <label for="imageX" class="block text-sm font-medium">ডানে/বামে সরান</label>
                        <input type="range" id="imageX" min="-200" max="200" step="1" value="0" class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer dark:bg-gray-700">
                    </div>
                    <div>
                        <label for="imageY" class="block text-sm font-medium">উপরে/নিচে সরান</label>
                        <input type="range" id="imageY" min="-200" max="200" step="1" value="0" class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer dark:bg-gray-700">
                    </div>
                </div>
            </div>

            <!-- Download Button -->
            <div class="mt-8">
                <button id="downloadBtn" class="w-full bg-indigo-600 text-white font-bold py-3 px-4 rounded-lg hover:bg-indigo-700 focus:outline-none focus:ring-4 focus:ring-indigo-500/50 transition-transform transform hover:scale-105 flex items-center justify-center gap-2">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor"><path fill-rule="evenodd" d="M3 17a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm3.293-7.707a1 1 0 011.414 0L9 10.586V3a1 1 0 112 0v7.586l1.293-1.293a1 1 0 111.414 1.414l-3 3a1 1 0 01-1.414 0l-3-3a1 1 0 010-1.414z" clip-rule="evenodd" /></svg>
                    ডাউনলোড করুন
                </button>
            </div>
        </div>

        <!-- Canvas/Preview Section -->
        <div class="lg:col-span-2 bg-white dark:bg-gray-800 p-4 rounded-2xl shadow-lg flex items-center justify-center">
            <canvas id="posterCanvas" class="rounded-lg shadow-inner w-full h-auto"></canvas>
        </div>
    </div>

    <script>
    document.addEventListener('DOMContentLoaded', () => {
        const canvas = document.getElementById('posterCanvas');
        const ctx = canvas.getContext('2d');
        const locationInput = document.getElementById('locationInput');
        const imageUpload = document.getElementById('imageUpload');
        const downloadBtn = document.getElementById('downloadBtn');
        const imageControls = document.getElementById('image-controls');
        const zoomSlider = document.getElementById('zoom');
        const xSlider = document.getElementById('imageX');
        const ySlider = document.getElementById('imageY');

        const canvasWidth = 800;
        const canvasHeight = 800;
        canvas.width = canvasWidth;
        canvas.height = canvasHeight;

        let userImage = null;
        let imageState = { zoom: 1, x: 0, y: 0 };

        const templateImage = new Image();
        templateImage.crossOrigin = "anonymous";
        templateImage.src = 'https://i.imgur.com/WVvwMe3.png'; 
        templateImage.onload = () => {
            drawCanvas();
        };
        templateImage.onerror = () => {
            console.error("Error loading the template image.");
        };

        function drawCanvas() {
            ctx.clearRect(0, 0, canvasWidth, canvasHeight);
            
            if (templateImage.complete && templateImage.naturalWidth !== 0) {
                ctx.drawImage(templateImage, 0, 0, canvasWidth, canvasHeight);
            }

            const circle = { x: 520, y: 280, radius: 180 };

            ctx.save();
            ctx.beginPath();
            ctx.arc(circle.x, circle.y, circle.radius, 0, Math.PI * 2, true);
            ctx.clip();

            if (userImage && userImage.complete) {
                const ratio = Math.max((circle.radius * 2) / userImage.width, (circle.radius * 2) / userImage.height);
                const finalZoom = ratio * imageState.zoom;
                const imgWidth = userImage.width * finalZoom;
                const imgHeight = userImage.height * finalZoom;
                const centerX = circle.x - imgWidth / 2;
                const centerY = circle.y - imgHeight / 2;
                ctx.drawImage(userImage, centerX + imageState.x, centerY + imageState.y, imgWidth, imgHeight);
            } else {
                ctx.fillStyle = 'rgba(255, 255, 255, 0.7)';
                ctx.fillRect(circle.x - circle.radius, circle.y - circle.radius, circle.radius * 2, circle.radius * 2);
                ctx.fillStyle = '#555';
                ctx.font = "24px 'Hind Siliguri'";
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText('আপনার ছবি দিন', circle.x, circle.y);
            }
            ctx.restore();

            ctx.textAlign = 'left';
            ctx.fillStyle = 'white';
            ctx.font = "bold 60px 'Hind Siliguri'";
            const locationText = locationInput.value;
            // Y-coordinate adjusted to 235 to move the text down slightly
            ctx.fillText(locationText, 40, 235);
        }

        locationInput.addEventListener('input', drawCanvas);

        imageUpload.addEventListener('change', (e) => {
            const file = e.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = (event) => {
                    userImage = new Image();
                    userImage.onload = () => {
                        imageControls.classList.remove('hidden');
                        zoomSlider.value = 1; xSlider.value = 0; ySlider.value = 0;
                        imageState = { zoom: 1, x: 0, y: 0 };
                        drawCanvas();
                    };
                    userImage.src = event.target.result;
                };
                reader.readAsDataURL(file);
            }
        });
        
        [zoomSlider, xSlider, ySlider].forEach(slider => {
            slider.addEventListener('input', () => {
                imageState.zoom = parseFloat(zoomSlider.value);
                imageState.x = parseInt(xSlider.value);
                imageState.y = parseInt(ySlider.value);
                drawCanvas();
            });
        });

        downloadBtn.addEventListener('click', () => {
            const link = document.createElement('a');
            link.download = 'poster-design.png';
            link.href = canvas.toDataURL('image/png');
            link.click();
        });

        // Initial draw
        drawCanvas();
    });
    </script>
</body>
</html>
