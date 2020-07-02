# Fishing Rod
>Our customers at vulnbox.net suffered a ransomware attack through phishing emails. Can you help us figure out why?

**DMARC (Domain-based Message Authentication & Conformance)** là phương pháp giúp giải quyết vấn đề của giao thức xác thực email.

Các vấn đề này gắn liền với khuôn khổ của **SPF** và **DKIM**. Đây là hai chính sách được hầu hết các nhà cung cấp hộp thư sử dụng để xử lý các email chứa virus hoặc mã độc, tin nhắn rác,...

**DMARC** giúp chống lại các cuộc tấn công sử dụng địa chỉ giả. (tham khảo thêm tại link [https://postmarkapp.com/guides/dmarc](https://postmarkapp.com/guides/dmarc) hoặc [https://wiki.matbao.net/dmarc-la-gi-huong-dan-cach-tao-dmarc-record-don-gian-nhat/](https://wiki.matbao.net/dmarc-la-gi-huong-dan-cach-tao-dmarc-record-don-gian-nhat/))

Domain `vulnbox.net` đã bị tấn công bằng email lừa đảo, cho nên chắc chắn việc cấu hình **SPF, DKIM hoặc DMARC** có vấn đề.
Chúng ta có thể kiếm tra tính hợp lệ của 3 giao thức này bằng công cụ online.

## Kiểm tra SPF
Sử dụng công cụ online [https://mxtoolbox.com/spf.aspx](https://mxtoolbox.com/spf.aspx), nhập domain là `vulnbox.net`

Kết quả:
```
v=spf1 include:spf.efwd.registrar-servers.com ~all - flag{wr0ng_SPF_
```

Vậy là chúng ta đã có 1 phần của flag `flag{wr0ng_SPF_`

## Kiểm tra DMARC
Sử dụng công cụ online [https://mxtoolbox.com/DMARC.aspx](https://mxtoolbox.com/DMARC.aspx), nhập domain là `vulnbox.net`

Kết quả:
```
v=DMARC1; p=quarantine; rua=mailto:reports@vunlbox.net; ruf=mailto:reports@dmarc.vulnbox.net; adkim=r; aspf=r; rf=afrf - 0r_DM4rC_
```
Ta có thêm một phần của flag nữa `0r_DM4rC_`

## Kiểm tra DKIM
Sử dụng công cụ online [https://mxtoolbox.com/dkim.aspx](https://mxtoolbox.com/dkim.aspx), nhập domain là `vulnbox.net`, selector là `dmarc`

Kết quả:
```
v=DKIM1;p=NG5kXzN2RW5fREsxbV9jT3VsZF9MRUBkX3QwX0VtQTFMX3BoMXNoSW5nX2E1M2ViZmZmM2YxMDk0YzA0fQ==
```
Dễ dàng đoán ra chuỗi `NG5kXzN2RW5fREsxbV9jT3VsZF9MRUBkX3QwX0VtQTFMX3BoMXNoSW5nX2E1M2ViZmZmM2YxMDk0YzA0fQ==` là chuỗi **Base64** 

Giải mã chuỗi trên ta được `4nd_3vEn_DK1m_cOuld_LE@d_t0_EmA1L_ph1shIng_a53ebfff3f1094c04}`

Nối 3 chuỗi trên lại => Flag: `flag{wr0ng_SPF_0r_DM4rC_4nd_3vEn_DK1m_cOuld_LE@d_t0_EmA1L_ph1shIng_a53ebfff3f1094c04}`