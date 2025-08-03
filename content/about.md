---
date : '2025-07-15T15:26:39+07:00'
title : 'About' 
---

<div class="about-container">
    <div class="profile-pic">
        <img src="https://i.postimg.cc/jjJh2CHN/avatar.jpg" alt="Ảnh đại diện Hà Sơn Bin">
    </div>
    <div class="profile-text">
        <h2>Chào bạn, mình là Hà Sơn Bin!</h2>
        <p>
            Hiện tại, mình đang là sinh viên chuyên ngành <strong>An toàn Thông tin</strong> tại trường <strong>Đại học Công nghệ Thông tin (UIT)</strong> - ĐHQG TP.HCM. Với niềm đam mê khám phá các lỗ hổng và xây dựng những hệ thống vững chắc, mình đã tạo ra blog này để chia sẻ những kiến thức mình học được cũng như các dự án cá nhân tâm đắc.
        </p>
        <p>
            Bạn có thể kết nối với mình qua: <a href="https://www.facebook.com/son.bin.ha.2025">Facebook</a>, <a href="https://www.youtube.com/@binzofficialentertainment5910">Youtube</a> hoặc liên hệ công việc qua email <a href="mailto:hasonbin123@gmail.com">hasonbin123@gmail.com</a>.
        </p>
    </div>
</div>

<h3>Một số kỹ năng của mình</h3>
<div class="skills">
    <span class="skill-tag">Network Security</span>
    <span class="skill-tag">Penetration Testing</span>
    <span class="skill-tag">Web Security</span>
    <span class="skill-tag">Python</span>
    <span class="skill-tag">Linux</span>
    <span class="skill-tag">C/C++</span>
    <span class="skill-tag">Burp Suite</span>
    </div>

<h3>Về trang web này</h3>
<p>
    Chào mừng bạn đến với blog cá nhân của mình! Đây là nơi mình ghi lại những kiến thức, ý tưởng thu nhận được từ các bài giảng chuyên ngành <b>An toàn Thông tin tại UIT</b>, đồng thời cũng là nơi mình giới thiệu các <b>dự án cá nhân</b> tâm đắc.
</p>
<p>
    Mục tiêu của mình là xây dựng một không gian học hỏi hữu ích cho các bạn cùng ngành và thể hiện niềm đam mê của mình với việc xây dựng và bảo vệ các hệ thống. Đừng ngần ngại khám phá và liên hệ với mình nếu bạn có bất kỳ câu hỏi hoặc muốn hợp tác trong các dự án nhé!
</p>

<style>
    .about-container {
        display: flex;
        align-items: center;
        gap: 30px; /* Khoảng cách giữa ảnh và chữ */
        margin-bottom: 40px;
    }
    .profile-pic {
        width: 150px;
        height: 150px;
        border-radius: 50%; /* Bo tròn ảnh */
        overflow: hidden; /* Đảm bảo ảnh không tràn ra ngoài border-radius */
    }
    .profile-pic img {
        width: 100%;
        height: 100%;
        object-fit: cover; /* Ảnh sẽ lấp đầy khung tròn mà không bị méo */
    }
    .profile-text {
        flex: 1;
    }
    .skills {
        display: flex;
        flex-wrap: wrap;
        gap: 10px;
        margin-top: 10px;
    }
    .skill-tag {
        background-color: #f0f0f0; /* Màu nền của tag */
        color: #333; /* Màu chữ của tag */
        padding: 5px 15px;
        border-radius: 20px;
        font-size: 0.9em;
    }
    .profile-text h2 {
        margin-top: 0;
        margin-bottom: 10px;
        font-size: 1.8em;
        color: #333; /* Màu tiêu đề */
    }
    .profile-text p {
        line-height: 1.6;
        color: #555; /* Màu chữ nội dung */
        margin-bottom: 15px;
    }
    .profile-text a {
        color: #007bff; /* Màu link */
        text-decoration: none;
        font-weight: bold;
    }
    .profile-text a:hover {
        text-decoration: underline;
    }

    /* Responsive cho điện thoại */
    @media (max-width: 600px) {
        .about-container {
            flex-direction: column;
            align-items: center;
            text-align: center;
        }
        .profile-pic {
            margin-bottom: 20px;
        }
    }
</style>