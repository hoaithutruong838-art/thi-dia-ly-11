<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Thi Địa Lý 11 - Bài 21: Kinh tế LB Nga</title>
    <style>
        :root { --primary: #2c3e50; --accent: #3498db; --danger: #e74c3c; --success: #27ae60; }
        body { font-family: 'Segoe UI', sans-serif; background: #f4f7f6; margin: 0; padding: 20px; display: flex; justify-content: center; }
        .container { width: 100%; max-width: 800px; background: white; padding: 30px; border-radius: 12px; box-shadow: 0 5px 15px rgba(0,0,0,0.1); }
        .hidden { display: none; }
        .login-box { text-align: center; }
        input[type="text"] { width: 90%; padding: 12px; margin: 10px 0; border: 1px solid #ddd; border-radius: 6px; }
        #timer { position: sticky; top: 10px; background: #fff3f3; padding: 10px; color: var(--danger); font-weight: bold; text-align: right; border: 1px solid #ffcccc; z-index: 100; margin-bottom: 20px; }
        .question-item { margin-bottom: 25px; padding: 15px; border-bottom: 1px solid #eee; }
        .options label { display: block; padding: 10px; background: #f9f9f9; margin: 5px 0; border-radius: 4px; cursor: pointer; }
        .options label:hover { background: #e3f2fd; }
        button { background: var(--accent); color: white; border: none; padding: 15px; border-radius: 6px; cursor: pointer; font-size: 18px; width: 100%; font-weight: bold; }
        #violationWarning { color: white; background: var(--danger); padding: 10px; position: fixed; bottom: 20px; left: 50%; transform: translateX(-50%); border-radius: 5px; z-index: 1000; font-weight: bold; }
    </style>
</head>
<body oncontextmenu="return false;" onselectstart="return false;">

<div class="container">
    <div id="login-screen" class="login-box">
        <h1>KIỂM TRA ĐỊA LÝ 11</h1>
        <h3>Bài 21: Kinh tế Liên bang Nga</h3>
        <input type="text" id="studentName" placeholder="Họ và Tên học sinh" required><br>
        <input type="text" id="studentClass" placeholder="Lớp (Ví dụ: 11A1)" required><br>
        <button onclick="startExam()">BẮT ĐẦU LÀM BÀI</button>
    </div>

    <div id="exam-screen" class="hidden">
        <div id="timer">Thời gian còn lại: <span id="timeDisplay">15:00</span></div>
        <div id="quiz-container"></div>
        <button onclick="confirmSubmit()">NỘP BÀI HOÀN TẤT</button>
    </div>

    <div id="result-screen" class="hidden">
        <h2 style="text-align:center;">KẾT QUẢ BÀI LÀM</h2>
        <div id="scoreDisplay" style="text-align: center; font-size: 24px; color: var(--primary); padding: 20px; background: #f1f8e9; border-radius: 8px;"></div>
        <div id="reviewArea" style="margin-top: 20px;"></div>
        <button onclick="location.reload()" style="background: var(--primary); margin-top: 20px;">THOÁT HỆ THỐNG</button>
    </div>
</div>

<div id="violationWarning" class="hidden"></div>

<script>
    // DỮ LIỆU CÂU HỎI TỪ TỆP CỦA BẠN
    const questions = [
        // NHẬN BIẾT
        { id: 1, q: "GDP năm 2020 của Liên bang Nga chiếm bao nhiêu % so với GDP toàn cầu?", a: ["Khoảng 1%", "Khoảng 1,7%", "Khoảng 2%", "Khoảng 2,7%"], c: "Khoảng 1,7%" },
        { id: 2, q: "Trồng trọt chiếm khoảng bao nhiêu % giá trị sản xuất nông nghiệp của Liên bang Nga?", a: ["Khoảng 20%", "Khoảng 30%", "Khoảng 40%", "Khoảng 50%"], c: "Khoảng 40%" },
        { id: 3, q: "Ngành đóng vai trò xương sống của nền kinh tế Liên Bang Nga là?", a: ["Năng lượng", "Công nghiệp", "Nông nghiệp", "Dịch vụ"], c: "Công nghiệp" },
        { id: 4, q: "Các ngành công nghiệp truyền thống của Liên bang Nga là?", a: ["Năng lượng; chế tạo máy, luyện kim đen...", "Năng lượng; luyện kim màu...", "Năng lượng; luyện kim đen, màu; khai thác vàng, kim cương, gỗ; sản xuất giấy, bột giấy", "Năng lượng; chế tạo máy; luyện kim đen, màu; khai thác vàng, kim cương, gỗ; sản xuất giấy, bột giấy"], c: "Năng lượng; chế tạo máy; luyện kim đen, màu; khai thác vàng, kim cương, gỗ; sản xuất giấy, bột giấy" },
        { id: 5, q: "Các vùng kinh tế quan trọng nhất của Liên bang Nga là?", a: ["Vùng Trung ương, vùng Trung tâm đất đen, vùng U-ran, vùng Viễn Đông", "Vùng Đông Bắc...", "Vùng Tây Bắc..."], c: "Vùng Trung ương, vùng Trung tâm đất đen, vùng U-ran, vùng Viễn Đông" },
        { id: 6, q: "Ở Nga, các ngành như năng lượng, chế tạo máy, luyện kim... được gọi là các ngành?", a: ["Mới", "Thủ công", "Truyền thống", "Hiện đại"], c: "Truyền thống" },
        { id: 7, q: "Điều kiện thuận lợi nhất trong sản xuất nông nghiệp của Liên Bang Nga?", a: ["Quỹ đất nông nghiệp lớn", "Khí hậu đa dạng", "Giáp biển", "Sông hồ lớn"], c: "Quỹ đất nông nghiệp lớn" },
        { id: 8, q: "Loại hình vận tải quan trọng nhất thúc đẩy phát triển vùng Đông Xi-bia?", a: ["Hàng không", "Đường sắt", "Đường biển", "Đường sông"], c: "Đường sắt" },
        { id: 9, q: "Hai trung tâm dịch vụ lớn nhất của Nga là?", a: ["Mát-xcơ-va và Vôn-ga-grát", "Xanh Pê-téc-bua và Vôn-ga-grát", "Vôn-ga-grát và Nô-vô-xi-biếc", "Mát-xcơ-va và Xanh Pê-téc-bua"], c: "Mát-xcơ-va và Xanh Pê-téc-bua" },
        { id: 10, q: "Diễn đàn kinh tế nhằm biến vùng Viễn Đông thành trung tâm kinh tế châu Á là?", a: ["APEC", "Diễn đàn kinh tế phương Đông (EEF)", "WEF Đông Á", "WTO"], c: "Diễn đàn kinh tế phương Đông (EEF)" },
        
        // THÔNG HIỂU
        { id: 11, q: "Ngành công nghiệp mũi nhọn mang lại tài chính lớn cho LB Nga là?", a: ["Hàng không", "Khai thác dầu khí", "Luyện kim", "Quốc phòng"], c: "Khai thác dầu khí" },
        { id: 12, q: "Ý nào không biểu hiện khó khăn của Nga sau khi Liên Xô tan rã?", a: ["Sản lượng giảm", "Vị thế thế giới giảm", "Tăng trưởng âm", "Đời sống nhân dân ổn định"], c: "Đời sống nhân dân ổn định" },
        { id: 13, q: "Nội dung cơ bản của chiến lược kinh tế mới từ năm 2000 là?", a: ["Đưa kinh tế thoát khỏi khủng hoảng", "Tiếp tục bao cấp", "Hạn chế ngoại giao", "Coi trọng châu Âu"], c: "Đưa kinh tế thoát khỏi khủng hoảng" },
        { id: 14, q: "Sau năm 2000 nền kinh tế của Liên Bang Nga đã?", a: ["Tăng lạm phát", "Tăng trưởng thần kỳ", "Vượt qua khủng hoảng, dần ổn định và đi lên", "Phát triển chậm lại"], c: "Vượt qua khủng hoảng, dần ổn định và đi lên" },
        { id: 15, q: "Ý nào không phải là thành tựu kinh tế của Nga sau năm 2000?", a: ["Sản lượng tăng", "Thanh toán xong nợ nước ngoài", "Xuất siêu tăng", "Đời sống nhân dân được nâng cao"], c: "Đời sống nhân dân được nâng cao" },
        { id: 16, q: "Ý nào đúng với họat động ngoại thương của Liên Bang Nga?", a: ["Xuất bằng nhập", "Hàng xuất là thủy sản", "Hàng nhập là dầu mỏ", "Tổng kim ngạch ngoại thương liên tục tăng"], c: "Tổng kim ngạch ngoại thương liên tục tăng" },
        { id: 17, q: "Điểm nào không đúng với kinh tế Nga?", a: ["Kinh tế đối ngoại khá quan trọng", "Dịch vụ phát triển mạnh", "Sản lượng nông nghiệp đứng hàng đầu thế giới", "Dầu khí là mũi nhọn"], c: "Sản lượng nông nghiệp đứng hàng đầu thế giới" },
        { id: 18, q: "Trong cải cách kinh tế sau năm 1990, Nga đã thực hiện giải pháp nào?", a: ["Đẩy mạnh tư hữu hoá", "Quốc hữu hoá", "Duy trì thủ công", "Tăng giá sản phẩm"], c: "Đẩy mạnh tư hữu hoá" },
        { id: 19, q: "Khó khăn lớn nhất đối với nông nghiệp của Liên bang Nga là?", a: ["Lãnh thổ đầm lầy, băng giá", "Khí hậu khắc nghiệt", "Dân số già", "Sông ngòi đóng băng"], c: "Lãnh thổ đầm lầy, băng giá" },
        { id: 20, q: "Một trong những thành tựu quan trọng đạt được về kinh tế của Liên Bang Nga là?", a: ["Gia tăng dân số nhanh", "Người di cư đông", "Phân hóa giàu nghèo", "Đời sống nhân dân đã được cải thiện"], c: "Đời sống nhân dân đã được cải thiện" }
    ];

    let currentQuiz = [];
    let timeLeft = 900;
    let violationCount = 0;
    let timerInterval;
    const scriptURL = 'DÁN_LINK_WEB_APP_GOOGLE_SCRIPT_CỦA_BẠN_VÀO_ĐÂY';

    function startExam() {
        const name = document.getElementById('studentName').value;
        const cls = document.getElementById('studentClass').value;
        if (!name || !cls) return alert("Vui lòng điền đủ thông tin!");
        document.getElementById('login-screen').classList.add('hidden');
        document.getElementById('exam-screen').classList.remove('hidden');
        currentQuiz = questions.sort(() => Math.random() - 0.5);
        renderQuiz();
        startTimer();
    }

    function renderQuiz() {
        const container = document.getElementById('quiz-container');
        container.innerHTML = currentQuiz.map((item, index) => `
            <div class="question-item">
                <p><strong>Câu ${index + 1}:</strong> ${item.q}</p>
                <div class="options">
                    ${item.a.map(opt => `<label><input type="radio" name="q${item.id}" value="${opt}"> ${opt}</label>`).join('')}
                </div>
            </div>
        `).join('');
    }

    function startTimer() {
        timerInterval = setInterval(() => {
            let m = Math.floor(timeLeft / 60);
            let s = timeLeft % 60;
            document.getElementById('timeDisplay').innerText = `${m}:${s < 10 ? '0' : ''}${s}`;
            if (timeLeft-- <= 0) submitExam();
        }, 1000);
    }

    document.addEventListener("visibilitychange", (https://script.google.com/macros/s/AKfycbzboWXAjivntbt1u9HtyUuqcrpRjYr5-l-w_vxAqoCQRVrcQfeyIdZvwMHJtyIy-SKpPw/exec) => {
        if (document.hidden && !document.getElementById('exam-screen').classList.contains('hidden')) {
            violationCount++;
            const warn = document.getElementById('violationWarning');
            warn.classList.remove('hidden');
            warn.innerText = `CẢNH BÁO: Bạn rời Tab lần ${violationCount}! (Tối đa 3 lần sẽ tự nộp)`;
            setTimeout(() => warn.classList.add('hidden'), 3000);
            if (violationCount >= 3) submitExam();
        }
    });

    function confirmSubmit() { if(confirm("Bạn có chắc muốn nộp bài?")) submitExam(); }

    async function submitExam() {
        clearInterval(timerInterval);
        let score = 0;
        let reviewHTML = "<h3>Chi tiết đáp án:</h3>";
        currentQuiz.forEach((item, index) => {
            const selected = document.querySelector(`input[name="q${item.id}"]:checked`)?.value;
            if (selected === item.c) {
                score++;
                reviewHTML += `<p style="color:green">Câu ${index+1}: Đúng</p>`;
            } else {
                reviewHTML += `<p style="color:red">Câu ${index+1}: Sai (Đáp án: ${item.c})</p>`;
            }
        });

        const finalPoints = (score / currentQuiz.length * 10).toFixed(1);
        document.getElementById('exam-screen').classList.add('hidden');
        document.getElementById('result-screen').classList.remove('hidden');
        document.getElementById('scoreDisplay').innerText = `KẾT QUẢ: ${finalPoints} / 10`;
        document.getElementById('reviewArea').innerHTML = reviewHTML;

        fetch(scriptURL, {
            method: 'POST',
            mode: 'no-cors',
            body: JSON.stringify({
                name: document.getElementById('studentName').value,
                className: document.getElementById('studentClass').value,
                score: finalPoints,
                violations: violationCount
            })
        });
    }
</script>
</body>
</html>
