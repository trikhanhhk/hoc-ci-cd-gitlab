# hoc-ci-cd-gitlab
Ghi chép học CI-CD git lab

Một số ghi chép tóm tắt lại những nội dung chính cần học khi tôi bắt đầu nghiên cứu devops, CI-CD trên gitlab

Trend phát triển và triển khai phần mềm nó gồm các bước như đây:
1. Tạo ra 1 image cho app của chúng ta
2. Chạy tất cả các test dựa vào image vừa được tạo (nếu pass thì tiếp tục đến bước 3 còn không thì fix đã 😃)
3. Đẩy image lên registry. (Nơi lưu trữ image) 
4. Deploy vào server của chúng ta
-> Đây là lý do Có CI-CD
   
### Nội dung tham khảo từ
[[Series] Học Docker, CICD từ cơ bản đến áp dụng vào thực tế](https://viblo.asia/s/jeZ103QgKWz) Của anh Mai Trung Đức. Mục đích ghi chép lại để mình ghi nhớ và note ra những phần mà bản thân mình thấy cần phải ghi nhớ.
