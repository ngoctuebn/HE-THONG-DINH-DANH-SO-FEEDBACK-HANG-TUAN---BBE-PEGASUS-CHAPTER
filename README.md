<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hệ Thống Định Danh Số - BBE Pegasus</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #f0f2f5; display: flex; justify-content: center; padding: 20px; }
        .container { background: white; padding: 30px; border-radius: 15px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); width: 100%; max-width: 500px; }
        h2 { color: #0056b3; text-align: center; }
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; }
        input, textarea { width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 5px; box-sizing: border-box; }
        button { width: 100%; padding: 12px; background-color: #0056b3; color: white; border: none; border-radius: 5px; cursor: pointer; font-size: 16px; font-weight: bold; }
        button:hover { background-color: #004494; }
    </style>
</head>
<body>
    <div class="container">
        <h2>PHẢN HỒI HÀNG TUẦN<br>BBE PEGASUS CHAPTER</h2>
        <div class="form-group">
            <label>Họ và Tên:</label>
            <input type="text" placeholder="Nhập tên của bạn...">
        </div>
        <div class="form-group">
            <label>Nội dung Feedback:</label>
            <textarea rows="4" placeholder="Viết phản hồi tại đây..."></textarea>
        </div>
        <button onclick="alert('Cảm ơn Tuệ! Phản hồi đã được gửi thành công (giả lập).')">GỬI PHẢN HỒI</button>
    </div>
</body>
</html>
