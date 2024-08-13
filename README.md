# Source
https://hub.docker.com/r/jwilder/nginx-proxy

https://blog.ssdnodes.com/blog/host-multiple-websites-docker-nginx/

Có thể xem thêm dùng cả letsencrypt: https://web.vnappmob.com/page/hosting-multiple-sites-or-applications-using-docker-and-nginx-reverse-proxy-with-letsencrypt-ssl-139

# Note
Để sử dụng nginx trong docker và phục vụ được các service nằm trong docker khác, thì cả 2 container phải sử dụng chung một network.

Tạo Network
`docker network create b_nginx_network`

nginx-proxy sẽ tự động đọc các docker container được khởi tạo nếu có dùng environment VIRTUAL_HOST do đọc qua docker.sock, và tự động khởi tạo để chạy mà không cần thêm file cấu hình riêng.
Có thể với các service dạng php muốn tối ưu nginx thì sẽ cấu hình thẳng 1 container nginx với các file cấu hình chi tiết vào cùng service docker-compose.yml rồi nginx-proxy tự nhận và điều hướng các luồng hoạt động thôi.

Chú ý file nginx.conf nếu có chỉnh sửa: phải có daemon off; ở cuối file, có 1 file default.conf được sinh ra trong /etc/nginx/conf.d có 1 số cấu hình như gzip có thể bị trùng lặp với file nginx.conf, nên nếu thay đổi phải check lại.

Bản nginx này sẽ không show trang default, mà thay vào đó là: "503 Service Temporarily Unavailable" - Miễn thấy trả về nginx là ok.

# Micro caching
Dùng để cache mọi request thành html và nginx sẽ trả về nội dung thay vì phải request đến web service (search đọc thêm để biết cấu hình chi tiết)

Để sử dụng cache phải thêm vào 1 file proxy-cache.conf vào folder conf.d với nội dung
```# proxy-cache.conf
proxy_cache_path  /var/lib/nginx/cache_all keys_zone=cache_all:20m levels=1:2 inactive=1d max_size=700m;
```
ở đây là file cache chung cho toàn bộ các trang, trang nào sử dụng thì thêm vào 1 file vhost trong thư mục vhost.d với tên file trùng tên domain (trùng name của VIRTUAL_HOST của domain đó) với nội dung:
`proxy_cache cache_all; proxy_cache_valid 200 60m; proxy_cache_valid 404 1m; proxy_buffering on; add_header X-Proxy-Cache $upstream_cache_status;`

ở đây 2 config này nghĩa là với các domain sử dụng proxy_cache tên là cache_all sẽ được cache vào folder proxy_cache_path với keys_zone tương ứng là cache_all. Muốn cấu hình riêng cho từng domain thì phải viết riêng folder và keys_zone riêng cho từng domain.

