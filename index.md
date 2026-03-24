---
layout: default
title: Trang chu
---

## Hello world

Tui là sinh viên năm 3 ngành An Toàn Thông Tin hệ cử nhân Tài Năng tại trường Đại Học Công Nghệ Thông Tin (UIT), định hướng nghề nghiệp là Network Infrastructure va Network Security. Trang này là nơi chia sẽ kiến thức, cv, và mong các nhà tuyển dụng để ý tới :>>

[Xem CV cua toi]({{ '/cv/' | relative_url }})

---

## Bai viet moi nhat

{% if site.posts.size > 0 %}
{% for post in site.posts %}
- [{{ post.title }}]({{ post.url | relative_url }}) - {{ post.date | date: "%d/%m/%Y" }}
{% endfor %}
{% else %}
Chua co bai viet nao. Hay tao bai dau tien trong thu muc `_posts`.
{% endif %}
