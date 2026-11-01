<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Đinh Trường</title>
    
    <link rel="icon" type="image/png" href="https://cdn-icons-png.flaticon.com/512/295/295118.png">
    <link rel="manifest" href="./manifest.json">
    
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="theme-color" content="#0f172a">
    
    <style>
        :root {
            --glass-bg: rgba(15, 23, 42, 0.75);
            --glass-border: rgba(255, 255, 255, 0.08);
            --text-main: #f8fafc;
            --text-muted: #94a3b8;
            --accent: #3b82f6;
            --bg-gradient: linear-gradient(135deg, #0f172a 0%, #1e1b4b 100%);
        }

        * {
            box-sizing: border-box;
            -webkit-tap-highlight-color: transparent;
        }

        body, html {
            font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            margin: 0;
            padding: 0;
            display: flex;
            height: 100vh;
            width: 100vw;
            overflow: hidden;
            background: var(--bg-gradient);
            color: var(--text-main);
            overscroll-behavior-y: contain; 
        }

        /* ================= LỚP PHỦ MỜ (BACKDROP) KHI MỞ DANH SÁCH ================= */
        .sidebar-backdrop {
            position: fixed;
            inset: 0;
            background-color: rgba(0, 0, 0, 0.4); /* Làm tối nhẹ màn hình ngoài */
            z-index: 800; /* Nằm dưới sidebar nhưng trên iframe */
            opacity: 0;
            visibility: hidden;
            transition: opacity 0.3s ease, visibility 0.3s ease;
        }

        .sidebar-backdrop.active {
            opacity: 1;
            visibility: visible;
        }

        /* ================= SIDEBAR ================= */
        .glass-panel {
            background: var(--glass-bg);
            backdrop-filter: blur(16px);
            -webkit-backdrop-filter: blur(16px);
            border-right: 1px solid var(--glass-border);
            box-shadow: 4px 0 24px rgba(0, 0, 0, 0.5);
        }

        .sidebar {
            width: 0; 
            display: flex;
            flex-direction: column;
            transition: width 0.3s cubic-bezier(0.16, 1, 0.3, 1);
            overflow: hidden; 
            white-space: nowrap; 
            position: absolute;
            height: 100%;
            left: 0;
            top: 0;
            z-index: 900;
        }

        .sidebar.open {
            width: 280px;
        }

        .web-list {
            list-style: none;
            padding: 30px 12px 20px 12px; 
            margin: 0;
            overflow-y: auto;
            flex-grow: 1;
            scrollbar-width: none; 
        }
        
        .web-list::-webkit-scrollbar {
            display: none; 
        }

        .web-list li {
            padding: 15px 20px;
            margin-bottom: 8px;
            border-radius: 12px;
            cursor: pointer;
            font-size: 15px;
            font-weight: 500;
            color: var(--text-muted);
            border: 1px solid transparent;
            transition: all 0.2s ease;
            display: flex;
            align-items: center;
            gap: 12px;
        }

        .web-list li::before {
            content: '';
            display: inline-block;
            width: 8px;
            height: 8px;
            border-radius: 50%;
            background-color: var(--text-muted);
            transition: all 0.3s ease;
        }

        .web-list li:hover, .web-list li:active {
            background: rgba(255, 255, 255, 0.05);
            color: var(--text-main);
        }

        .web-list li.active {
            background: rgba(59, 130, 246, 0.15);
            border: 1px solid rgba(59, 130, 246, 0.3);
            color: var(--text-main);
        }
        
        .web-list li.active::before {
            background-color: var(--accent);
            box-shadow: 0 0 8px var(--accent);
        }

        /* ================= KHUNG NỘI DUNG ================= */
        .content {
            flex-grow: 1;
            width: 100%;
            height: 100%;
            display: flex;
            position: relative;
            background-color: #ffffff;
        }

        iframe {
            width: 100%;
            height: 100%;
            border: none;
            z-index: 1;
        }

        .overlay-screen {
            position: absolute;
            inset: 0;
            background: var(--bg-gradient);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 10;
            transition: opacity 0.3s ease;
        }

        .spinner {
            width: 36px;
            height: 36px;
            border: 3px solid rgba(255,255,255,0.1);
            border-radius: 50%;
            border-top-color: var(--accent);
            animation: spin 1s ease-in-out infinite;
            margin-bottom: 16px;
        }

        @keyframes spin { to { transform: rotate(360deg); } }

        .placeholder-text {
            color: var(--text-muted);
            font-size: 15px;
            text-align: center;
        }

        /* ================= NÚT TRỢ LÝ ẢO ================= */
        .fab-btn {
            position: fixed;
            top: 20px;
            left: 20px; 
            z-index: 9999;
            background: rgba(15, 23, 42, 0.85);
            backdrop-filter: blur(8px);
            -webkit-backdrop-filter: blur(8px);
            border: 1px solid rgba(255,255,255,0.2);
            color: white;
            border-radius: 50%;
            width: 44px;
            height: 44px;
            display: flex;
            align-items: center;
            justify-content: center;
            box-shadow: 0 4px 15px rgba(0,0,0,0.3);
            transition: background-color 0.2s, transform 0.1s; 
            touch-action: none; 
            cursor: grab;
        }

        .fab-btn:active {
            cursor: grabbing;
            transform: scale(0.95);
        }

        .fab-btn svg {
            width: 22px;
            height: 22px;
            fill: currentColor;
            pointer-events: none; 
        }

        .fab-btn.is-open {
            background: rgba(220, 38, 38, 0.85);
            border-color: rgba(239, 68, 68, 0.5);
        }

        /* Nút Tải Ứng Dụng */
        #installAppBtn {
            display: none; 
            position: fixed;
            bottom: 30px;
            right: 30px;
            background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);
            color: white;
            border: none;
            padding: 12px 20px;
            border-radius: 50px;
            font-size: 14px;
            font-weight: 600;
            box-shadow: 0 6px 15px rgba(37, 99, 235, 0.4);
            z-index: 1000;
            align-items: center;
            gap: 8px;
        }
    </style>
</head>
<body>

    <!-- LỚP PHỦ MỜ BÊN NGOÀI ĐỂ CLICK ĐÓNG -->
    <div id="sidebarBackdrop" class="sidebar-backdrop active"></div>

    <div class="sidebar glass-panel open" id="sidebar">
        <ul class="web-list" id="webList"></ul>
    </div>

    <div id="fabBtn" class="fab-btn is-open">
        <svg id="icon-menu" viewBox="0 0 24 24"><path d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z"/></svg>
    </div>

    <div class="content">
        <div id="overlayScreen" class="overlay-screen">
            <div id="loadingSpinner" class="spinner"></div>
            <div id="placeholderText" class="placeholder-text">Đang đồng bộ dữ liệu...</div>
        </div>
        <iframe id="webFrame" src="about:blank" title="Giao diện Web"></iframe>
    </div>

    <button id="installAppBtn" style="display: none;">
        <svg style="width:16px;height:16px;" viewBox="0 0 24 24" fill="white"><path d="M19 9h-4V3H9v6H5l7 7 7-7zM5 18v2h14v-2H5z"/></svg>
        Cài Đặt
    </button>

    <script>
        // ================= KÉO THẢ TRỢ LÝ ẢO =================
        const fabBtn = document.getElementById('fabBtn');
        let isDragging = false;
        let hasMoved = false; 
        let currentX, currentY, initialX, initialY;
        let xOffset = 0;
        let yOffset = 0;

        const savedPos = JSON.parse(localStorage.getItem('fabPosition'));
        if (savedPos) {
            xOffset = savedPos.x;
            yOffset = savedPos.y;
            setTranslate(xOffset, yOffset, fabBtn);
        }

        fabBtn.addEventListener("touchstart", dragStart, { passive: false });
        fabBtn.addEventListener("touchend", dragEnd, { passive: false });
        fabBtn.addEventListener("touchmove", drag, { passive: false });
        
        fabBtn.addEventListener("mousedown", dragStart);
        document.addEventListener("mouseup", dragEnd);
        document.addEventListener("mousemove", drag);

        function dragStart(e) {
            initialX = (e.type === "touchstart" ? e.touches[0].clientX : e.clientX) - xOffset;
            initialY = (e.type === "touchstart" ? e.touches[0].clientY : e.clientY) - yOffset;
            
            if (e.target === fabBtn || fabBtn.contains(e.target)) {
                isDragging = true;
                hasMoved = false; 
            }
        }

        function drag(e) {
            if (isDragging) {
                e.preventDefault(); 
                currentX = (e.type === "touchmove" ? e.touches[0].clientX : e.clientX) - initialX;
                currentY = (e.type === "touchmove" ? e.touches[0].clientY : e.clientY) - initialY;
                
                const rect = fabBtn.getBoundingClientRect();
                const maxX = window.innerWidth - rect.width - 20; 
                const maxY = window.innerHeight - rect.height - 20;
                
                if(currentX > maxX) currentX = maxX;
                if(currentX < -20) currentX = -20; 
                if(currentY > maxY) currentY = maxY;
                if(currentY < -20) currentY = -20;

                xOffset = currentX;
                yOffset = currentY;
                
                hasMoved = true;
                setTranslate(currentX, currentY, fabBtn);
            }
        }

        function dragEnd(e) {
            if(isDragging) {
                initialX = currentX;
                initialY = currentY;
                isDragging = false;
                localStorage.setItem('fabPosition', JSON.stringify({x: xOffset, y: yOffset}));
            }
        }

        function setTranslate(xPos, yPos, el) {
            el.style.transform = `translate3d(${xPos}px, ${yPos}px, 0)`;
        }

        // ================= XỬ LÝ MENU & ẤN RA NGOÀI ĐỂ ĐÓNG =================
        const sidebar = document.getElementById('sidebar');
        const iconMenu = document.getElementById('icon-menu');
        const sidebarBackdrop = document.getElementById('sidebarBackdrop');

        // Hàm Đóng/Mở Sidebar chung
        function toggleSidebar(forceClose = false) {
            if (forceClose) {
                sidebar.classList.remove('open');
            } else {
                sidebar.classList.toggle('open');
            }
            
            // Cập nhật trạng thái của Backdrop (lớp mờ) và Icon
            if (sidebar.classList.contains('open')) {
                sidebarBackdrop.classList.add('active');
                fabBtn.classList.add('is-open');
                iconMenu.innerHTML = `<path d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z"/>`;
            } else {
                sidebarBackdrop.classList.remove('active');
                fabBtn.classList.remove('is-open');
                iconMenu.innerHTML = `<path d="M3 18h18v-2H3v2zm0-5h18v-2H3v2zm0-7v2h18V6H3z"/>`;
            }
        }

        // Bấm nút Trợ lý ảo
        fabBtn.addEventListener('click', (e) => {
            if (hasMoved) {
                hasMoved = false;
                return; 
            }
            toggleSidebar();
        });

        // ĐẢM BẢO 100%: Bấm vào vùng mờ bên ngoài sẽ Đóng danh sách
        sidebarBackdrop.addEventListener('click', () => {
            toggleSidebar(true);
        });

        // ================= FETCH API & RENDER LIST =================
        const overlayScreen = document.getElementById('overlayScreen');
        const placeholderText = document.getElementById('placeholderText');
        const loadingSpinner = document.getElementById('loadingSpinner');

        document.addEventListener("DOMContentLoaded", () => {
            const url = "https://script.google.com/macros/s/AKfycbzU2-uP0jUUia8w74BBQ7U78OUn3d_SE7cEd_LBZJbq2b2g4HCJyFRAOLKMU9p2zb0F/exec";
            fetch(url, {
                method: "POST",
                headers: { "Content-Type": "text/plain;charset=utf-8" },
                body: JSON.stringify({ "sheet": "Home", "action": "get" })
            })
            .then(res => res.json())
            .then(data => {
                loadingSpinner.style.display = 'none';
                if (data.status === "success" && data.all_data?.Home) {
                    renderList(data.all_data.Home);
                    placeholderText.innerText = "Chọn một ứng dụng từ Menu";
                } else {
                    placeholderText.innerText = "Dữ liệu không hợp lệ.";
                }
            })
            .catch(err => {
                loadingSpinner.style.display = 'none';
                placeholderText.innerText = "Lỗi kết nối máy chủ!";
            });
        });

        function renderList(rows) {
            const listContainer = document.getElementById('webList');
            for (let i = 1; i < rows.length; i++) {
                const tenWeb = rows[i].A;
                const urlWeb = rows[i].B;
                if (!tenWeb) continue; 

                const listItem = document.createElement('li');
                listItem.innerText = tenWeb;
                
                listItem.addEventListener('click', function() {
                    listContainer.querySelectorAll('li').forEach(item => item.classList.remove('active'));
                    this.classList.add('active');
                    
                    toggleSidebar(true); // Đóng sidebar sau khi chọn
                    loadWeb(tenWeb, urlWeb);
                });
                listContainer.appendChild(listItem);
            }
        }

        function loadWeb(name, url) {
            const iframe = document.getElementById('webFrame');
            document.title = name;
            
            if (url && (url.startsWith('http://') || url.startsWith('https://'))) {
                overlayScreen.style.opacity = '0';
                setTimeout(() => overlayScreen.style.display = 'none', 300);
                iframe.src = url;
            } else {
                iframe.src = "about:blank"; 
                overlayScreen.style.display = 'flex';
                overlayScreen.style.opacity = '1';
                loadingSpinner.style.display = 'none';
                placeholderText.innerText = `URL lỗi: ${url}`;
            }
        }

        // ================= SERVICE WORKER =================
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', () => {
                navigator.serviceWorker.register('./sw.js').catch(()=>{});
            });
        }
        let deferredPrompt;
        const installBtn = document.getElementById('installAppBtn');
        window.addEventListener('beforeinstallprompt', (e) => {
            e.preventDefault();
            deferredPrompt = e;
            installBtn.style.display = 'flex';
        });
        installBtn.addEventListener('click', async () => {
            if (deferredPrompt) {
                deferredPrompt.prompt();
                const { outcome } = await deferredPrompt.userChoice;
                deferredPrompt = null;
                installBtn.style.display = 'none';
            }
        });
        window.addEventListener('appinstalled', () => installBtn.style.display = 'none');
    </script>
</body>
</html>
