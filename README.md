addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request));
});

/**
 * Respond to the request
 * @param {Request} request
 */
async function handleRequest(request) {
  if (request.method === 'POST') {
    const formData = await request.formData();
    const passwordCount = parseInt(formData.get('count')) || 5;
    const length = parseInt(formData.get('length')) || 32;
    const includeSpecialChars = formData.get('special') === 'true';
    const category = formData.get('category') || 'numbers-mixed';

    const passwords = generatePasswords(passwordCount, length, includeSpecialChars, category);
    const responseHTML = getFormHTML(passwords, passwordCount, length, includeSpecialChars, category);

    return new Response(responseHTML, {
      headers: { 'content-type': 'text/html' },
    });
  } else {
    return new Response(getFormHTML(), {
      headers: { 'content-type': 'text/html' },
    });
  }
}

/**
 * Generate random passwords based on given criteria
 * @param {number} count
 * @param {number} length
 * @param {boolean} includeSpecialChars
 * @param {string} category
 * @returns {string[]}
 */
function generatePasswords(count, length, includeSpecialChars, category) {
  const charsetLower = 'abcdefghijklmnopqrstuvwxyz';
  const charsetUpper = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
  const charsetNumbers = '0123456789';
  const charsetSpecial = '!@#$%^&*()_+-=[]{}|;:,.<>?';

  let charset = '';
  switch (category) {
    case 'uppercase':
      charset = charsetUpper;
      break;
    case 'lowercase':
      charset = charsetLower;
      break;
    case 'mixed':
      charset = charsetLower + charsetUpper;
      break;
    case 'numbers':
      charset = charsetNumbers;
      break;
    case 'numbers-uppercase':
      charset = charsetNumbers + charsetUpper;
      break;
    case 'numbers-lowercase':
      charset = charsetNumbers + charsetLower;
      break;
    case 'numbers-mixed':
      charset = charsetNumbers + charsetLower + charsetUpper;
      break;
    default:
      charset = charsetLower + charsetUpper + charsetNumbers;
      break;
  }

  if (includeSpecialChars) {
    charset += charsetSpecial;
  }

  const passwords = [];
  for (let i = 0; i < count; i++) {
    passwords.push(generatePassword(length, charset));
  }

  return passwords;
}

/**
 * Generate a random password from given charset
 * @param {number} length
 * @param {string} charset
 * @returns {string}
 */
function generatePassword(length, charset) {
  let password = '';
  for (let i = 0; i < length; i++) {
    const randomIndex = Math.floor(Math.random() * charset.length);
    password += charset[randomIndex];
  }
  return password;
}

/**
 * Get HTML form for user input and optionally display passwords
 * @param {string[]} [passwords]
 * @param {number} [passwordCount]
 * @param {number} [length]
 * @param {boolean} [includeSpecialChars]
 * @param {string} [category]
 * @returns {string}
 */
function getFormHTML(passwords = [], passwordCount = 5, length = 32, includeSpecialChars = false, category = 'numbers-mixed') {
  let passwordsHTML = '';
  if (passwords.length > 0) {
    passwordsHTML += `
      <div class="password-list">
        <h2>生成的密码</h2>
        <textarea id="passwords-container" readonly>${passwords.join('\n')}</textarea>
      </div>
    `;
  }

  return `
    <!DOCTYPE html>
    <html lang="zh-CN">
    <head>
      <meta charset="UTF-8">
      <title>随机密码生成工具</title>
      <style>
        body {
          font-family: Arial, sans-serif;
          background-color: #f4f4f9;
          margin: 0;
          padding: 20px;
          display: flex;
          flex-direction: column;
          align-items: center;
        }
        h1 {
          color: #333;
        }
        form {
          background: #fff;
          padding: 20px;
          border-radius: 10px;
          box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
          max-width: 500px;
          width: 100%;
        }
        .form-group {
          display: flex;
          flex-direction: column;
          margin-bottom: 20px;
        }
        .form-group label {
          margin-bottom: 5px;
        }
        .form-group input, .form-group select {
          padding: 10px;
          border: 1px solid #ddd;
          border-radius: 5px;
        }
        .inline-group {
          display: flex;
          justify-content: space-between;
        }
        .inline-group .form-group {
          flex: 1;
          margin-right: 10px;
        }
        .inline-group .form-group:last-child {
          margin-right: 0;
        }
        input[type="submit"] {
          background: #5cb85c;
          color: #fff;
          border: none;
          cursor: pointer;
          padding: 10px;
          border-radius: 5px;
        }
        input[type="submit"]:hover {
          background: #4cae4c;
        }
        .password-list {
          background: #fff;
          padding: 20px;
          border-radius: 10px;
          box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
          max-width: 500px;
          width: 100%;
          margin-top: 20px;
          text-align: center;
        }
        .password-list textarea {
          width: 100%;
          height: 150px;
          font-family: 'Courier New', Courier, monospace;
          border: 1px solid #ddd;
          border-radius: 5px;
          padding: 10px;
          resize: none;
          text-align: center;
        }
        .copy-button {
          margin-top: 10px;
          padding: 10px;
          border: none;
          background-color: #007bff;
          color: #fff;
          cursor: pointer;
          border-radius: 5px;
        }
        .copy-button:hover {
          background-color: #0056b3;
        }
        .footer {
          margin-top: 20px;
          text-align: center;
          color: #777;
          max-width: 500px;
          width: 100%;
          padding: 20px;
          border-radius: 10px;
          box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
          background: #fff;
        }
        .footer h2 {
          margin-bottom: 10px;
        }
        .footer p {
          margin: 5px 0;
        }
        .clear-button {
          background-color: #dc3545;
          color: white;
          padding: 10px;
          border: none;
          border-radius: 5px;
          cursor: pointer;
          margin-top: 10px;
        }
        .clear-button:hover {
          background-color: #c82333;
        }
      </style>
      <script>
        function copyToClipboard() {
          const passwordsContainer = document.getElementById('passwords-container');
          passwordsContainer.select();
          document.execCommand('copy');
          alert('密码已复制到剪贴板');
        }

        function clearForm() {
          document.getElementById('count').value = '';
          document.getElementById('length').value = '';
          document.getElementById('category').value = 'numbers-mixed';
          document.getElementById('special').value = 'false';
        }
      </script>
    </head>
    <body>
      <h1>密码生成器</h1>
      <form method="post">
        <div class="inline-group">
          <div class="form-group">
            <label for="count">生成密码数量:</label>
            <input type="number" id="count" name="count" min="5" max="50" placeholder="最低5 - 最高50" required>
          </div>
          <div class="form-group">
            <label for="length">密码长度:</label>
            <input type="number" id="length" name="length" min="8" max="32" placeholder="最低8 - 最高32" required>
          </div>
        </div>
        <div class="form-group">
          <label for="category">密码类别:</label>
          <select id="category" name="category">
            <option value="random" ${category === 'random' ? 'selected' : ''}>随机</option>
            <option value="uppercase" ${category === 'uppercase' ? 'selected' : ''}>纯大写字母</option>
            <option value="lowercase" ${category === 'lowercase' ? 'selected' : ''}>纯小写字母</option>
            <option value="mixed" ${category === 'mixed' ? 'selected' : ''}>大小写字母混合</option>
            <option value="numbers" ${category === 'numbers' ? 'selected' : ''}>纯数字</option>
            <option value="numbers-uppercase" ${category === 'numbers-uppercase' ? 'selected' : ''}>数字和大写字母</option>
            <option value="numbers-lowercase" ${category === 'numbers-lowercase' ? 'selected' : ''}>数字和小写字母</option>
            <option value="numbers-mixed" ${category === 'numbers-mixed' ? 'selected' : ''}>数字和混合大小写字母</option>
          </select>
        </div>
        <div class="form-group">
          <label for="special">是否包含特殊字符:</label>
          <select id="special" name="special">
            <option value="true" ${includeSpecialChars ? 'selected' : ''}>包含特殊字符</option>
            <option value="false" ${!includeSpecialChars ? 'selected' : ''}>不包含特殊字符</option>
          </select>
        </div>
        <div class="form-group" style="text-align: center;">
          <input type="submit" value="生成密码">
          <button type="button" class="clear-button" onclick="clearForm()">清空</button>
          ${passwords.length > 0 ? '<button type="button" class="copy-button" onclick="copyToClipboard()">一键复制</button>' : ''}
        </div>
      </form>
      ${passwordsHTML}
      <div class="footer">
        <h2>说明</h2>
        <p>随机字符、随机数字、随机密码在线生成工具</p>
        <p>高强度随机密码生成器：可自定义生成随机数字、大小写字母、特殊字符的随机密码生成工具</p>
        <p>随机字符生成器：支持纯数字、纯字母、纯字母（大写、小写）、数字和字母（混合）、 数字和字母（大写、小写）、混合特殊字符等多种组合，自定义输出的长度和批量生成数量</p>
        <p>可任意组合需要的字符进行随机密码字符生成，可以作为随机密码密钥生成器用于项目测试使用，也可以自行决定使用用途</p>
      </div>
    </body>
    </html>
  `;
}