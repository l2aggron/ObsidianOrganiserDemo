---
created: <% tp.date.now("YYYY-MM-DD HH:mm:ss (Z), dddd") %>
tags:
  - personalTask
Приоритет: "100"
---
Зачем:: <% tp.file.cursor(1) %>

- [ ] <% tp.file.cursor(0) %> 🏷️ ➕ <% tp.date.now("YYYY-MM-DD") %>
