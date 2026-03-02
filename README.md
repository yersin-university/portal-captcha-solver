## Vấn đề: Gõ Captcha mỗi lần đăng nhập Cổng thông tin thật phiền phức!

Nếu bạn là sinh viên và thường xuyên phải truy cập vào trang Yersin Portal (Cổng thông tin đào tạo), chắc hẳn bạn sẽ thấy một điều rất quen thuộc nhưng cũng không kém phần phiền toái: **Luôn phải nhập mã Captcha bằng tay mỗi lần đăng nhập**.

- Tốn thời gian thao tác mỗi khi cần vào xem lịch học, điểm thi.
- Đôi khi mã Captcha khó nhìn, nhập sai phải tải lại và gõ lại từ đầu.
- Làm gián đoạn trải nghiệm truy cập nhanh chóng.

---

## Giải pháp: Script "Auto Solve Captcha" tự động điền mã

Bằng cách sử dụng **Tampermonkey** – tiện ích mở rộng cho phép chạy các script tùy biến trên trình duyệt – mình ([@kamedev02](https://github.com/kamedev02)) đã viết một đoạn **script tự động** giúp bạn giải quyết vấn đề này (mặc dù đôi lúc vẫn có sai sót):

- Tự động **trích xuất hình ảnh Captcha** trên trang đăng nhập.
- Gửi ảnh đến server AI/Proxy để **nhận diện mặt chữ siêu tốc**.
- **Tự động điền kết quả** vào ô nhập mã mà bạn không cần đụng tay.

---

## Tính năng nổi bật của script

| Tính năng                           | Hỗ trợ |
|------------------------------------|--------|
| Tự động nhận diện ảnh Captcha      | ✅     |
| Kết nối qua Proxy API để giải mã   | ✅     |
| Tự động điền (Auto-fill) vào input | ✅     |
| Hoạt động ngầm, không cần UI phụ   | ✅     |

---

## Cài đặt Tampermonkey và sử dụng

### 1. Cài đặt tiện ích Tampermonkey

Link cài đặt: [https://www.tampermonkey.net](https://www.tampermonkey.net)

### 2. Thêm script mới

- Vào **Tampermonkey Dashboard** → Chọn tab **“+” (Create a new script)**.
- Xóa đoạn code mặc định và **dán đoạn code bên dưới** vào:

```js
// ==UserScript==
// @name         Auto Solve Captcha
// @namespace    http://tampermonkey.net/
// @version      0.0.1
// @description  Auto Solve Yersin Portal Captcha
// @author       KameDev
// @match        https://portal.yersin.edu.vn/Login
// @grant        GM_xmlhttpRequest
// @connect      captcha-proxy.nhocbuifa.workers.dev
// ==/UserScript==

(function () {
    'use strict';

    function sleep(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }

    function imageToBase64() {
        const img = document.querySelector('#CaptchaImage');
        const canvas = document.createElement('canvas');
        canvas.width = img.naturalWidth;
        canvas.height = img.naturalHeight;
        const ctx = canvas.getContext('2d');
        ctx.drawImage(img, 0, 0);
        const base64 = canvas.toDataURL('image/png');
        return base64.split(',')[1];
    }

    function sendCaptchaToProxy(base64Image) {
        return new Promise((resolve, reject) => {
            GM_xmlhttpRequest({
                method: 'POST',
                url: 'https://captcha-proxy.nhocbuifa.workers.dev',
                headers: {
                    'Content-Type': 'application/json'
                },
                data: JSON.stringify({ image: base64Image }),
                onload: function (res) {
                    try {
                        const data = JSON.parse(res.responseText);
                        if (data.solution) {
                            resolve(data.solution);
                        } else {
                            console.error("Server returned error:", data);
                            reject(data);
                        }
                    } catch (e) {
                        reject(e);
                    }
                },
                onerror: function (err) {
                    reject(err);
                }
            });
        });
    }

    async function main() {
        const base64Image = imageToBase64();
        if (!base64Image) return;

        try {
            const solution = await sendCaptchaToProxy(base64Image);

            const input = document.querySelector("#CaptchaInputText");
            if (input) {
                input.value = solution;
            } else {
                console.warn('Captcha input not found');
            }
        } catch (error) {
            console.error('Error during captcha processing:', error);
        }
    }

    window.addEventListener('load', () => {
        main();
    });
})();
```

- Nhấn File → Save (hoặc dùng phím tắt `Ctrl + S` / `Cmd + S`).

### 3. Trải nghiệm
- Truy cập vào trang đăng nhập: [Yersin Portal Login](https://portal.yersin.edu.vn/Login).

- Chờ khoảng 1-2 giây để hệ thống tự động quét ảnh và trả về kết quả. Mã Captcha sẽ được **tự động điền vào ô trống**!

- Bạn chỉ việc điền tài khoản, mật khẩu (nếu trình duyệt chưa tự lưu) và bấm Đăng nhập.

---

## Ưu điểm
- **Tiết kiệm thời gian**: Đăng nhập vào cổng thông tin nhanh hơn hẳn.

- **Độ chính xác cao**: Thông qua proxy giải mã tự động.

- **Chạy hoàn toàn ngầm**: Không hiển thị popup hay nút bấm làm rối giao diện.

- **Nhẹ & an toàn**: Script chỉ lấy ảnh Captcha và gửi đi, không thu thập bất cứ dữ liệu cá nhân hay mật khẩu nào của bạn.

---

## Kết luận
Việc tự động hóa những tác vụ lặp đi lặp lại nhàm chán như gõ Captcha giúp chúng ta có trải nghiệm học tập và làm việc trên môi trường số tốt hơn. Chỉ với một đoạn script Tampermonkey nhỏ, việc đăng nhập Yersin Portal của bạn giờ đây đã trở nên mượt mà và hiện đại hơn rất nhiều!

---

## Liên hệ
- Blog: [levanson065@gmail.com](mailto:levanson065@gmail.com)

- GitHub: [@kamedev02](https://github.com/kamedev02)

- YouTube: [@kamedev](https://www.youtube.com/@kamedev)

- Discord: [Khoa CNTT - YersinUni](https://discord.gg/BgXZ5At4Nb)

> Mọi đóng góp hoặc ý tưởng cải tiến đều được hoan nghênh!
