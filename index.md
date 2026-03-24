---
layout: default
title: Trang chu
---

## Xin chao, minh la Pham Thanh An

Minh la sinh vien nam 3 nganh Information Security (Honors Program) tai University of Information Technology (UIT), dang tap trung vao Network Infrastructure va Network Security. Trang nay la noi minh tong hop CV, cac du an tieu bieu va bai viet chia se kinh nghiem hoc tap - thuc hanh.

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
