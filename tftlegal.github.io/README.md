# GIT
```
git add .
git commit -m "feat: Set new hugo.toml"
git push -uf origin main
```
# Claude

# План трансформации Hugo → Static Cyberpunk

Я создам полностью статичный сайт-визитку в стиле киберпанк. Вот структура и все файлы:

## 📁 Структура проекта

```
tftlegal.github.io/
│
├── index.html
├── styles.css
├── script.js
├── favicon.svg
├── robots.txt
├── sitemap.xml
│
└── assets/
    ├── og-image.jpg
    └── logo.svg
```

---

## 📄 **index.html**

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>TFT Legal — Digital Law & Cyber Security</title>
    <meta name="description" content="Юридическое сопровождение цифрового бизнеса, Web3, Blockchain, IT и международных проектов.">
    <meta name="keywords" content="TFT Legal, Blockchain, Web3, Crypto, IT Law, Smart Contracts, Compliance">
    <meta name="author" content="TFT Legal">
    
    <!-- Theme -->
    <meta name="theme-color" content="#050505">
    <meta name="color-scheme" content="dark">
    
    <!-- Open Graph -->
    <meta property="og:type" content="website">
    <meta property="og:site_name" content="TFT Legal">
    <meta property="og:title" content="TFT Legal — Digital Law & Cyber Security">
    <meta property="og:description" content="Юридическое сопровождение цифрового бизнеса, Web3, Blockchain, IT и международных проектов">
    <meta property="og:image" content="https://tftlegal.github.io/assets/og-image.jpg">
    <meta property="og:url" content="https://tftlegal.github.io">
    
    <!-- Twitter Card -->
    <meta name="twitter:card" content="summary_large_image">
    <meta name="twitter:title" content="TFT Legal — Digital Law & Cyber Security">
    <meta name="twitter:description" content="Юридическое сопровождение цифрового бизнеса">
    <meta name="twitter:image" content="https://tftlegal.github.io/assets/og-image.jpg">
    
    <!-- Favicon -->
    <link rel="icon" type="image/svg+xml" href="favicon.svg">
    <link rel="alternate icon" href="favicon.ico">
    
    <!-- Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;700;900&family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    
    <!-- Styles -->
    <link rel="stylesheet" href="styles.css">
    
    <!-- Canonical -->
    <link rel="canonical" href="https://tftlegal.github.io">
</head>

<body>

<!-- Background Effects -->
<div class="grid"></div>
<div class="noise"></div>

<!-- Header -->
<header id="header">
    <div class="container">
        <a class="logo" href="#hero" aria-label="TFT Legal Home">
            <span class="logo-tft">TFT</span><span class="logo-legal">LEGAL</span>
        </a>
        
        <nav role="navigation" aria-label="Main Navigation">
            <a href="#about">О нас</a>
            <a href="#services">Услуги</a>
            <a href="#advantages">Преимущества</a>
            <a href="#contacts">Контакты</a>
        </nav>
        
        <button class="mobile-menu" id="mobileMenuBtn" aria-label="Toggle Menu">
            <span></span>
            <span></span>
            <span></span>
        </button>
    </div>
</header>

<!-- Mobile Navigation -->
<div class="mobile-nav" id="mobileNav">
    <a href="#about">О нас</a>
    <a href="#services">Услуги</a>
    <a href="#advantages">Преимущества</a>
    <a href="#contacts">Контакты</a>
</div>

<!-- Hero Section -->
<section id="hero">
    <div class="container hero">
        <div class="hero-left fade-in">
            <div class="label">
                DIGITAL • BLOCKCHAIN • WEB3
            </div>
            
            <h1>
                LEGAL
                <span class="gradient-text">FOR THE</span>
                DIGITAL ERA
            </h1>
            
            <p class="hero-description">
                TFT Legal предоставляет юридическое сопровождение технологических компаний, 
                криптопроектов, международного бизнеса и цифровых платформ.
            </p>
            
            <div class="hero-buttons">
                <a href="#contacts" class="btn-primary">Связаться</a>
                <a href="#services" class="btn-secondary">Услуги</a>
            </div>
        </div>
        
        <div class="hero-right fade-in-delay">
            <div class="terminal">
                <div class="terminal-header">
                    <div class="terminal-dots">
                        <span></span>
                        <span></span>
                        <span></span>
                    </div>
                    <div class="terminal-title">SYSTEM STATUS</div>
                </div>
                <div class="terminal-body">
<pre class="terminal-text">
<span class="cyan">$</span> <span class="cmd">system.check</span> --status

<span class="key">Connection</span> ............ <span class="green">ONLINE</span>
<span class="key">Jurisdiction</span> ......... <span class="cyan">GLOBAL</span>
<span class="key">Compliance</span> ........... <span class="green">VERIFIED</span>
<span class="key">Blockchain</span> ........... <span class="purple">ACTIVE</span>
<span class="key">Smart Contracts</span> ...... <span class="green">ENABLED</span>
<span class="key">Digital Assets</span> ....... <span class="cyan">PROTECTED</span>

<span class="blink">▊</span>
</pre>
                </div>
            </div>
        </div>
    </div>
</section>

<!-- About Section -->
<section id="about">
    <div class="container">
        <h2 class="section-title fade-in-up">О <span class="gradient-text">компании</span></h2>
        
        <div class="glass fade-in-up">
            <div class="glass-content">
                <p>
                    <strong>TFT Legal</strong> — команда специалистов, сопровождающая цифровые проекты 
                    на международном рынке.
                </p>
                
                <p>
                    Мы консультируем компании в области <span class="highlight">Blockchain</span>, 
                    <span class="highlight">Web3</span>, криптовалют, интеллектуальной собственности, 
                    IT, международного корпоративного права и комплаенса.
                </p>
                
                <div class="about-features">
                    <div class="feature-item">
                        <div class="feature-icon">⚡</div>
                        <div class="feature-text">Инновационные технологии</div>
                    </div>
                    <div class="feature-item">
                        <div class="feature-icon">🌐</div>
                        <div class="feature-text">Глобальная юрисдикция</div>
                    </div>
                    <div class="feature-item">
                        <div class="feature-icon">🔒</div>
                        <div class="feature-text">Конфиденциальность</div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</section>

<!-- Services Section -->
<section id="services">
    <div class="container">
        <h2 class="section-title fade-in-up">Наши <span class="gradient-text">услуги</span></h2>
        
        <div class="cards">
            <article class="card fade-in-up" style="animation-delay: 0.1s">
                <div class="card-icon">⛓️</div>
                <h3>Blockchain & Web3</h3>
                <p>
                    Полное юридическое сопровождение криптопроектов, токенизации, DAO, NFT 
                    и цифровых активов.
                </p>
                <ul class="card-list">
                    <li>Регистрация криптобирж</li>
                    <li>ICO/IDO/IEO консалтинг</li>
                    <li>Смарт-контракты</li>
                    <li>DeFi протоколы</li>
                </ul>
            </article>
            
            <article class="card fade-in-up" style="animation-delay: 0.2s">
                <div class="card-icon">💻</div>
                <h3>IT Law</h3>
                <p>
                    Договоры, лицензии, SaaS, Software Development, Privacy Policy, GDPR.
                </p>
                <ul class="card-list">
                    <li>Лицензионные соглашения</li>
                    <li>Защита ПО</li>
                    <li>GDPR/Privacy</li>
                    <li>API & SaaS договоры</li>
                </ul>
            </article>
            
            <article class="card fade-in-up" style="animation-delay: 0.3s">
                <div class="card-icon">🏢</div>
                <h3>Corporate Law</h3>
                <p>
                    Международные компании, регистрация, сопровождение бизнеса, инвестиции 
                    и корпоративные сделки.
                </p>
                <ul class="card-list">
                    <li>Открытие компаний</li>
                    <li>M&A сделки</li>
                    <li>Инвестиционные раунды</li>
                    <li>Корпоративное управление</li>
                </ul>
            </article>
            
            <article class="card fade-in-up" style="animation-delay: 0.4s">
                <div class="card-icon">✅</div>
                <h3>Compliance & AML</h3>
                <p>
                    AML, KYC, Due Diligence, риск-анализ, аудит цифровых платформ.
                </p>
                <ul class="card-list">
                    <li>AML/KYC процедуры</li>
                    <li>Комплаенс-аудит</li>
                    <li>FATF регуляции</li>
                    <li>Due Diligence</li>
                </ul>
            </article>
        </div>
    </div>
</section>

<!-- Advantages Section -->
<section id="advantages">
    <div class="container">
        <h2 class="section-title fade-in-up">Наши <span class="gradient-text">преимущества</span></h2>
        
        <div class="stats">
            <div class="stat fade-in-up" style="animation-delay: 0.1s">
                <div class="stat-value">10+</div>
                <div class="stat-label">Лет опыта</div>
                <div class="stat-glow"></div>
            </div>
            
            <div class="stat fade-in-up" style="animation-delay: 0.2s">
                <div class="stat-value">Global</div>
                <div class="stat-label">Международные проекты</div>
                <div class="stat-glow"></div>
            </div>
            
            <div class="stat fade-in-up" style="animation-delay: 0.3s">
                <div class="stat-value">24/7</div>
                <div class="stat-label">Поддержка клиентов</div>
                <div class="stat-glow"></div>
            </div>
            
            <div class="stat fade-in-up" style="animation-delay: 0.4s">
                <div class="stat-value">Web3</div>
                <div class="stat-label">Цифровая экспертиза</div>
                <div class="stat-glow"></div>
            </div>
        </div>
    </div>
</section>

<!-- Contacts Section -->
<section id="contacts">
    <div class="container">
        <h2 class="section-title fade-in-up">Связаться <span class="gradient-text">с нами</span></h2>
        
        <div class="glass fade-in-up">
            <div class="contacts-grid">
                <div class="contact-item">
                    <div class="contact-icon">📧</div>
                    <div class="contact-info">
                        <div class="contact-label">Email</div>
                        <a href="mailto:hello@tftlegal.com" class="contact-link">hello@tftlegal.com</a>
                    </div>
                </div>
                
                <div class="contact-item">
                    <div class="contact-icon">💬</div>
                    <div class="contact-info">
                        <div class="contact-label">Telegram</div>
                        <a href="https://t.me/tftlegal" target="_blank" rel="noopener" class="contact-link">@tftlegal</a>
                    </div>
                </div>
                
                <div class="contact-item">
                    <div class="contact-icon">🐙</div>
                    <div class="contact-info">
                        <div class="contact-label">GitHub</div>
                        <a href="https://github.com/tftlegal" target="_blank" rel="noopener" class="contact-link">github.com/tftlegal</a>
                    </div>
                </div>
            </div>
            
            <div class="cta-box">
                <p class="cta-text">Готовы обсудить ваш проект?</p>
                <a href="mailto:hello@tftlegal.com" class="btn-primary">Написать нам</a>
            </div>
        </div>
    </div>
</section>

<!-- Footer -->
<footer>
    <div class="container">
        <div class="footer-content">
            <div class="footer-left">
                <div class="logo">
                    <span class="logo-tft">TFT</span><span class="logo-legal">LEGAL</span>
                </div>
                <p class="footer-tagline">Digital Legal Solutions</p>
            </div>
            
            <div class="footer-right">
                <p>&copy; 2024 TFT Legal. All rights reserved.</p>
            </div>
        </div>
    </div>
</footer>

<!-- Scripts -->
<script src="script.js"></script>

</body>
</html>
```

---

## 🎨 **styles.css**

```css
/* ===========================
   TFT LEGAL CYBERPUNK THEME
   Author: TFT Legal Team
   Version: 1.0
=========================== */

/* ===== CSS VARIABLES ===== */
:root {
    /* Colors */
    --bg: #050505;
    --bg-alt: #0a0a0f;
    --panel: rgba(15, 15, 20, 0.65);
    --glass: rgba(20, 25, 35, 0.4);
    
    /* Neon Colors */
    --cyan: #00F5FF;
    --purple: #A000FF;
    --blue: #5FD8FF;
    --pink: #FF00FF;
    --green: #00FF88;
    
    /* Text */
    --text: #E8F4FF;
    --text-muted: #92A5C2;
    --text-dark: #6B7A94;
    
    /* Borders */
    --border: rgba(0, 245, 255, 0.22);
    --border-purple: rgba(160, 0, 255, 0.22);
    
    /* Shadows */
    --shadow-cyan: 0 0 8px rgba(0, 245, 255, 0.35), 0 0 18px rgba(0, 245, 255, 0.22);
    --shadow-purple: 0 0 8px rgba(160, 0, 255, 0.35), 0 0 18px rgba(160, 0, 255, 0.22);
    --shadow-strong: 0 0 20px rgba(0, 245, 255, 0.5), 0 0 40px rgba(0, 245, 255, 0.3);
    
    /* Radius */
    --radius: 18px;
    --radius-sm: 12px;
    --radius-lg: 24px;
    
    /* Spacing */
    --container-width: 1180px;
    --section-padding: 110px;
}

/* ===== RESET ===== */
*, *::before, *::after {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

html {
    scroll-behavior: smooth;
    overflow-x: hidden;
}

body {
    background: var(--bg);
    color: var(--text);
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    overflow-x: hidden;
    line-height: 1.7;
    position: relative;
    min-height: 100vh;
}

/* ===== SELECTION ===== */
::selection {
    background: var(--cyan);
    color: var(--bg);
}

::-moz-selection {
    background: var(--cyan);
    color: var(--bg);
}

/* ===== SCROLLBAR ===== */
::-webkit-scrollbar {
    width: 10px;
}

::-webkit-scrollbar-track {
    background: var(--bg);
}

::-webkit-scrollbar-thumb {
    background: var(--cyan);
    border-radius: 10px;
    box-shadow: var(--shadow-cyan);
}

::-webkit-scrollbar-thumb:hover {
    background: var(--blue);
}

/* ===== BACKGROUND EFFECTS ===== */
body::before {
    content: "";
    position: fixed;
    inset: 0;
    background: 
        radial-gradient(circle at 20% 30%, rgba(0, 255, 255, 0.12), transparent 40%),
        radial-gradient(circle at 80% 20%, rgba(180, 0, 255, 0.15), transparent 45%),
        radial-gradient(circle at 50% 90%, rgba(0, 255, 255, 0.08), transparent 35%);
    pointer-events: none;
    z-index: -4;
}

/* Grid */
.grid {
    position: fixed;
    inset: 0;
    background-image: 
        linear-gradient(rgba(0, 255, 255, 0.06) 1px, transparent 1px),
        linear-gradient(90deg, rgba(0, 255, 255, 0.06) 1px, transparent 1px);
    background-size: 50px 50px;
    opacity: 0.35;
    z-index: -3;
    pointer-events: none;
}

/* Scanlines */
.noise {
    position: fixed;
    inset: 0;
    pointer-events: none;
    background: repeating-linear-gradient(
        0deg,
        rgba(255, 255, 255, 0.02),
        rgba(255, 255, 255, 0.02) 2px,
        transparent 3px,
        transparent 6px
    );
    animation: scan 10s linear infinite;
    opacity: 0.4;
    z-index: -2;
}

@keyframes scan {
    from { transform: translateY(0); }
    to { transform: translateY(100px); }
}

/* ===== LAYOUT ===== */
.container {
    width: min(var(--container-width), 92%);
    margin: 0 auto;
}

section {
    padding: var(--section-padding) 0;
    position: relative;
}

/* ===== TYPOGRAPHY ===== */
h1, h2, h3, h4, h5, h6 {
    font-family: 'Orbitron', monospace;
    letter-spacing: 2px;
    font-weight: 700;
}

h2 {
    font-size: clamp(36px, 5vw, 48px);
    margin-bottom: 50px;
}

.section-title {
    color: var(--text);
    text-align: center;
}

.gradient-text {
    background: linear-gradient(135deg, var(--cyan), var(--purple));
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    display: inline-block;
}

.highlight {
    color: var(--cyan);
    font-weight: 600;
}

p {
    color: var(--text-muted);
    font-size: 16px;
}

strong {
    color: var(--text);
    font-weight: 600;
}

/* ===== HEADER ===== */
header {
    position: fixed;
    width: 100%;
    top: 0;
    z-index: 999;
    backdrop-filter: blur(16px);
    background: rgba(5, 5, 5, 0.75);
    border-bottom: 1px solid var(--border);
    transition: all 0.3s ease;
}

header.scrolled {
    background: rgba(5, 5, 5, 0.95);
    box-shadow: 0 4px 30px rgba(0, 0, 0, 0.5);
}

header .container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 80px;
}

/* Logo */
.logo {
    text-decoration: none;
    font-size: clamp(24px, 3vw, 30px);
    font-family: 'Orbitron', monospace;
    font-weight: 900;
    display: flex;
    align-items: center;
    gap: 2px;
}

.logo-tft {
    color: white;
}

.logo-legal {
    color: var(--cyan);
    text-shadow: var(--shadow-cyan);
}

/* Navigation */
nav {
    display: flex;
    gap: 34px;
}

nav a {
    color: var(--text);
    text-decoration: none;
    transition: all 0.3s ease;
    position: relative;
    font-weight: 500;
    font-size: 15px;
}

nav a:hover {
    color: var(--cyan);
    text-shadow: 0 0 12px var(--cyan);
}

nav a::after {
    content: "";
    position: absolute;
    bottom: -6px;
    left: 0;
    width: 0;
    height: 2px;
    background: var(--cyan);
    transition: width 0.3s ease;
    box-shadow: var(--shadow-cyan);
}

nav a:hover::after {
    width: 100%;
}

/* Mobile Menu Button */
.mobile-menu {
    display: none;
    flex-direction: column;
    gap: 5px;
    background: none;
    border: none;
    cursor: pointer;
    padding: 8px;
}

.mobile-menu span {
    width: 25px;
    height: 3px;
    background: var(--cyan);
    border-radius: 3px;
    transition: all 0.3s ease;
    box-shadow: var(--shadow-cyan);
}

.mobile-menu.active span:nth-child(1) {
    transform: rotate(45deg) translate(7px, 7px);
}

.mobile-menu.active span:nth-child(2) {
    opacity: 0;
}

.mobile-menu.active span:nth-child(3) {
    transform: rotate(-45deg) translate(7px, -7px);
}

/* Mobile Navigation */
.mobile-nav {
    position: fixed;
    top: 80px;
    left: 0;
    right: 0;
    background: rgba(5, 5, 5, 0.98);
    backdrop-filter: blur(20px);
    border-bottom: 1px solid var(--border);
    padding: 20px;
    display: none;
    flex-direction: column;
    gap: 20px;
    z-index: 998;
    opacity: 0;
    transform: translateY(-20px);
    transition: all 0.3s ease;
}

.mobile-nav.active {
    display: flex;
    opacity: 1;
    transform: translateY(0);
}

.mobile-nav a {
    color: var(--text);
    text-decoration: none;
    font-size: 18px;
    padding: 12px 16px;
    border-left: 3px solid transparent;
    transition: all 0.3s ease;
}

.mobile-nav a:hover {
    color: var(--cyan);
    border-left-color: var(--cyan);
    background: var(--glass);
}

/* ===== HERO SECTION ===== */
#hero {
    min-height: 100vh;
    display: flex;
    align-items: center;
    padding-top: 80px;
}

.hero {
    display: grid;
    grid-template-columns: 1.2fr 1fr;
    gap: 70px;
    align-items: center;
}

.hero-left {
    max-width: 700px;
}

.label {
    display: inline-block;
    color: var(--cyan);
    border: 1px solid var(--cyan);
    padding: 10px 18px;
    border-radius: 40px;
    margin-bottom: 30px;
    box-shadow: var(--shadow-cyan);
    font-size: 13px;
    font-weight: 600;
    letter-spacing: 1px;
}

.hero h1 {
    font-size: clamp(48px, 8vw, 74px);
    line-height: 1.05;
    margin-bottom: 30px;
    color: white;
}

.hero h1 span {
    display: block;
}

.hero-description {
    max-width: 620px;
    font-size: clamp(16px, 2vw, 18px);
    margin-bottom: 40px;
    line-height: 1.7;
}

/* Buttons */
.hero-buttons {
    display: flex;
    gap: 20px;
    flex-wrap: wrap;
}

.btn-primary,
.btn-secondary {
    text-decoration: none;
    padding: 16px 34px;
    border-radius: 50px;
    transition: all 0.35s ease;
    font-weight: 600;
    font-size: 15px;
    display: inline-block;
    text-align: center;
}

.btn-primary {
    background: var(--cyan);
    color: black;
    box-shadow: var(--shadow-cyan);
    border: none;
}

.btn-primary:hover {
    transform: translateY(-4px);
    box-shadow: 0 0 25px var(--cyan), 0 0 50px var(--cyan);
}

.btn-secondary {
    border: 2px solid var(--purple);
    color: var(--purple);
    background: transparent;
}

.btn-secondary:hover {
    background: var(--purple);
    color: white;
    box-shadow: var(--shadow-purple);
    transform: translateY(-4px);
}

/* Terminal */
.terminal {
    background: var(--glass);
    border: 1px solid var(--border);
    border-radius: var(--radius);
    overflow: hidden;
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.4);
    backdrop-filter: blur(10px);
}

.terminal-header {
    background: rgba(10, 10, 15, 0.8);
    padding: 12px 16px;
    display: flex;
    align-items: center;
    gap: 12px;
    border-bottom: 1px solid var(--border);
}

.terminal-dots {
    display: flex;
    gap: 6px;
}

.terminal-dots span {
    width: 12px;
    height: 12px;
    border-radius: 50%;
    display: block;
}

.terminal-dots span:nth-child(1) {
    background: #ff5f56;
    box-shadow: 0 0 8px #ff5f56;
}

.terminal-dots span:nth-child(2) {
    background: #ffbd2e;
    box-shadow: 0 0 8px #ffbd2e;
}

.terminal-dots span:nth-child(3) {
    background: #27c93f;
    box-shadow: 0 0 8px #27c93f;
}

.terminal-title {
    font-family: 'Orbitron', monospace;
    font-size: 13px;
    color: var(--cyan);
    letter-spacing: 1px;
}

.terminal-body {
    padding: 24px;
}

.terminal-text {
    font-family: 'Courier New', monospace;
    font-size: 14px;
    line-height: 1.8;
    color: var(--text-muted);
}

.terminal-text .cyan {
    color: var(--cyan);
}

.terminal-text .purple {
    color: var(--purple);
}

.terminal-text .green {
    color: var(--green);
}

.terminal-text .key {
    color: var(--text);
}

.terminal-text .cmd {
    color: var(--blue);
}

.blink {
    animation: blink 1s infinite;
    color: var(--cyan);
}

@keyframes blink {
    0%, 49% { opacity: 1; }
    50%, 100% { opacity: 0; }
}

/* ===== GLASS EFFECT ===== */
.glass {
    background: var(--glass);
    border: 1px solid var(--border);
    border-radius: var(--radius);
    padding: 40px;
    backdrop-filter: blur(12px);
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
}

.glass-content p {
    margin-bottom: 20px;
    font-size: 17px;
    line-height: 1.8;
}

.glass-content p:last-child {
    margin-bottom: 0;
}

/* About Features */
.about-features {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 20px;
    margin-top: 30px;
}

.feature-item {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 16px;
    background: rgba(0, 245, 255, 0.05);
    border: 1px solid var(--border);
    border-radius: var(--radius-sm);
    transition: all 0.3s ease;
}

.feature-item:hover {
    background: rgba(0, 245, 255, 0.1);
    transform: translateY(-4px);
    box-shadow: var(--shadow-cyan);
}

.feature-icon {
    font-size: 24px;
}

.feature-text {
    color: var(--text);
    font-size: 14px;
    font-weight: 500;
}

/* ===== SERVICES CARDS ===== */
.cards {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 30px;
}

.card {
    background: var(--glass);
    border: 1px solid var(--border);
    border-radius: var(--radius);
    padding: 32px;
    transition: all 0.4s ease;
    backdrop-filter: blur(12px);
    position: relative;
    overflow: hidden;
}

.card::before {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 3px;
    background: linear-gradient(90deg, var(--cyan), var(--purple));
    opacity: 0;
    transition: opacity 0.3s ease;
}

.card:hover::before {
    opacity: 1;
}

.card:hover {
    transform: translateY(-8px);
    border-color: var(--cyan);
    box-shadow: var(--shadow-strong);
}

.card-icon {
    font-size: 48px;
    margin-bottom: 20px;
    display: block;
}

.card h3 {
    font-size: 24px;
    margin-bottom: 16px;
    color: var(--text);
    font-family: 'Orbitron', monospace;
}

.card p {
    margin-bottom: 20px;
    line-height: 1.7;
}

.card-list {
    list-style: none;
    margin: 0;
    padding: 0;
}

.card-list li {
    padding: 8px 0 8px 20px;
    position: relative;
    color: var(--text-muted);
    font-size: 14px;
}

.card-list li::before {
    content: "▸";
    position: absolute;
    left: 0;
    color: var(--cyan);
    font-weight: bold;
}

/* ===== STATS ===== */
.stats {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
    gap: 40px;
}

.stat {
    text-align: center;
    padding: 40px 20px;
    background: var(--glass);
    border: 1px solid var(--border);
    border-radius: var(--radius);
    position: relative;
    overflow: hidden;
    backdrop-filter: blur(12px);
    transition: all 0.4s ease;
}

.stat:hover {
    transform: translateY(-8px);
    border-color: var(--purple);
    box-shadow: var(--shadow-purple);
}

.stat-value {
    font-size: clamp(48px, 6vw, 64px);
    font-family: 'Orbitron', monospace;
    font-weight: 900;
    background: linear-gradient(135deg, var(--cyan), var(--purple));
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    margin-bottom: 10px;
}

.stat-label {
    color: var(--text-muted);
    font-size: 16px;
    font-weight: 500;
}

.stat-glow {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    height: 3px;
    background: linear-gradient(90deg, var(--cyan), var(--purple));
    opacity: 0;
    transition: opacity 0.3s ease;
}

.stat:hover .stat-glow {
    opacity: 1;
    box-shadow: 0 0 20px var(--cyan);
}

/* ===== CONTACTS ===== */
.contacts-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 30px;
    margin-bottom: 40px;
}

.contact-item {
    display: flex;
    align-items: flex-start;
    gap: 16px;
    padding: 20px;
    background: rgba(0, 245, 255, 0.03);
    border: 1px solid var(--border);
    border-radius: var(--radius-sm);
    transition: all 0.3s ease;
}

.contact-item:hover {
    background: rgba(0, 245, 255, 0.08);
    border-color: var(--cyan);
    transform: translateX(8px);
}

.contact-icon {
    font-size: 32px;
}

.contact-label {
    color: var(--text-dark);
    font-size: 13px;
    text-transform: uppercase;
    letter-spacing: 1px;
    margin-bottom: 4px;
}

.contact-link {
    color: var(--cyan);
    text-decoration: none;
    font-weight: 600;
    font-size: 16px;
    transition: all 0.3s ease;
}

.contact-link:hover {
    text-shadow: var(--shadow-cyan);
}

/* CTA Box */
.cta-box {
    text-align: center;
    padding: 40px;
    background: rgba(160, 0, 255, 0.05);
    border: 1px solid var(--border-purple);
    border-radius: var(--radius);
    margin-top: 40px;
}

.cta-text {
    font-size: 20px;
    color: var(--text);
    margin-bottom: 24px;
    font-weight: 600;
}

/* ===== FOOTER ===== */
footer {
    background: rgba(5, 5, 5, 0.95);
    border-top: 1px solid var(--border);
    padding: 40px 0;
    margin-top: 60px;
}

.footer-content {
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 20px;
}

.footer-tagline {
    color: var(--text-dark);
    font-size: 14px;
    margin-top: 8px;
}

.footer-right p {
    color: var(--text-dark);
    font-size: 14px;
}

/* ===== ANIMATIONS ===== */
@keyframes fadeInUp {
    from {
        opacity: 0;
        transform: translateY(30px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

@keyframes fadeIn {
    from {
        opacity: 0;
    }
    to {
        opacity: 1;
    }
}

.fade-in {
    animation: fadeIn 1s ease-out;
}

.fade-in-delay {
    animation: fadeIn 1s ease-out 0.3s both;
}

.fade-in-up {
    animation: fadeInUp 0.8s ease-out;
}

/* ===== RESPONSIVE ===== */
@media (max-width: 968px) {
    :root {
        --section-padding: 70px;
    }
    
    nav {
        display: none;
    }
    
    .mobile-menu {
        display: flex;
    }
    
    .hero {
        grid-template-columns: 1fr;
        gap: 50px;
    }
    
    .hero-right {
        order: -1;
    }
    
    .cards {
        grid-template-columns: 1fr;
    }
    
    .stats {
        grid-template-columns: repeat(2, 1fr);
    }
}

@media (max-width: 640px) {
    header .container {
        height: 70px;
    }
    
    .hero h1 {
        font-size: 42px;
    }
    
    h2 {
        font-size: 32px;
        margin-bottom: 30px;
    }
    
    .glass {
        padding: 24px;
    }
    
    .card {
        padding: 24px;
    }
    
    .stats {
        grid-template-columns: 1fr;
        gap: 20px;
    }
    
    .contacts-grid {
        grid-template-columns: 1fr;
    }
    
    .hero-buttons {
        flex-direction: column;
    }
    
    .btn-primary,
    .btn-secondary {
        width: 100%;
    }
    
    .footer-content {
        flex-direction: column;
        text-align: center;
    }
}

/* ===== ACCESSIBILITY ===== */
@media (prefers-reduced-motion: reduce) {
    *,
    *::before,
    *::after {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
    }
}

/* Focus visible */
*:focus-visible {
    outline: 2px solid var(--cyan);
    outline-offset: 4px;
}

/* ===== UTILITY CLASSES ===== */
.text-center {
    text-align: center;
}

.mt-1 { margin-top: 10px; }
.mt-2 { margin-top: 20px; }
.mt-3 { margin-top: 30px; }
.mb-1 { margin-bottom: 10px; }
.mb-2 { margin-bottom: 20px; }
.mb-3 { margin-bottom: 30px; }
```

---

## ⚙️ **script.js**

```javascript
// ===========================
// TFT LEGAL - JavaScript
// ===========================

(function() {
    'use strict';

    // ===== Mobile Menu Toggle =====
    const mobileMenuBtn = document.getElementById('mobileMenuBtn');
    const mobileNav = document.getElementById('mobileNav');
    
    if (mobileMenuBtn && mobileNav) {
        mobileMenuBtn.addEventListener('click', function() {
            this.classList.toggle('active');
            mobileNav.classList.toggle('active');
            document.body.style.overflow = mobileNav.classList.contains('active') ? 'hidden' : '';
        });

        // Close mobile menu when clicking on a link
        const mobileLinks = mobileNav.querySelectorAll('a');
        mobileLinks.forEach(link => {
            link.addEventListener('click', () => {
                mobileMenuBtn.classList.remove('active');
                mobileNav.classList.remove('active');
                document.body.style.overflow = '';
            });
        });
    }

    // ===== Header Scroll Effect =====
    const header = document.getElementById('header');
    let lastScroll = 0;

    window.addEventListener('scroll', () => {
        const currentScroll = window.pageYOffset;
        
        if (currentScroll > 100) {
            header.classList.add('scrolled');
        } else {
            header.classList.remove('scrolled');
        }
        
        lastScroll = currentScroll;
    });

    // ===== Smooth Scroll for Anchor Links =====
    document.querySelectorAll('a[href^="#"]').forEach(anchor => {
        anchor.addEventListener('click', function(e) {
            const href = this.getAttribute('href');
            
            if (href === '#') return;
            
            e.preventDefault();
            const target = document.querySelector(href);
            
            if (target) {
                const headerHeight = header.offsetHeight;
                const targetPosition = target.getBoundingClientRect().top + window.pageYOffset - headerHeight;
                
                window.scrollTo({
                    top: targetPosition,
                    behavior: 'smooth'
                });
            }
        });
    });

    // ===== Intersection Observer for Animations =====
    const observerOptions = {
        threshold: 0.1,
        rootMargin: '0px 0px -50px 0px'
    };

    const observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                entry.target.style.opacity = '1';
                entry.target.style.transform = 'translateY(0)';
            }
        });
    }, observerOptions);

    // Observe all elements with fade-in-up class
    document.querySelectorAll('.fade-in-up').forEach(el => {
        el.style.opacity = '0';
        el.style.transform = 'translateY(30px)';
        el.style.transition = 'opacity 0.8s ease-out, transform 0.8s ease-out';
        observer.observe(el);
    });

    // ===== Terminal Text Animation =====
    const terminalText = document.querySelector('.terminal-text');
    if (terminalText) {
        const originalHTML = terminalText.innerHTML;
        terminalText.innerHTML = '';
        
        let i = 0;
        const speed = 20;
        
        function typeWriter() {
            if (i < originalHTML.length) {
                terminalText.innerHTML += originalHTML.charAt(i);
                i++;
                setTimeout(typeWriter, speed);
            }
        }
        
        // Start typing animation when terminal is in viewport
        const terminalObserver = new IntersectionObserver((entries) => {
            entries.forEach(entry => {
                if (entry.isIntersecting) {
                    setTimeout(typeWriter, 500);
                    terminalObserver.unobserve(entry.target);
                }
            });
        }, { threshold: 0.5 });
        
        terminalObserver.observe(terminalText);
    }

    // ===== Stats Counter Animation =====
    const stats = document.querySelectorAll('.stat-value');
    
    const animateStats = (entries, observer) => {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                const stat = entry.target;
                const text = stat.textContent;
                
                // Only animate if it's a number
                if (!isNaN(parseInt(text))) {
                    const target = parseInt(text);
                    let current = 0;
                    const increment = target / 50;
                    
                    const counter = setInterval(() => {
                        current += increment;
                        if (current >= target) {
                            stat.textContent = text;
                            clearInterval(counter);
                        } else {
                            stat.textContent = Math.floor(current) + '+';
                        }
                    }, 30);
                }
                
                observer.unobserve(stat);
            }
        });
    };
    
    const statsObserver = new IntersectionObserver(animateStats, {
        threshold: 0.5
    });
    
    stats.forEach(stat => statsObserver.observe(stat));

    // ===== Active Navigation Highlight =====
    const sections = document.querySelectorAll('section[id]');
    const navLinks = document.querySelectorAll('nav a[href^="#"]');
    
    function highlightNav() {
        const scrollY = window.pageYOffset;
        
        sections.forEach(section => {
            const sectionHeight = section.offsetHeight;
            const sectionTop = section.offsetTop - 100;
            const sectionId = section.getAttribute('id');
            
            if (scrollY > sectionTop && scrollY <= sectionTop + sectionHeight) {
                navLinks.forEach(link => {
                    link.classList.remove('active');
                    if (link.getAttribute('href') === `#${sectionId}`) {
                        link.classList.add('active');
                    }
                });
            }
        });
    }
    
    window.addEventListener('scroll', highlightNav);

    // ===== Card Hover 3D Effect =====
    const cards = document.querySelectorAll('.card');
    
    cards.forEach(card => {
        card.addEventListener('mousemove', (e) => {
            const rect = card.getBoundingClientRect();
            const x = e.clientX - rect.left;
            const y = e.clientY - rect.top;
            
            const centerX = rect.width / 2;
            const centerY = rect.height / 2;
            
            const rotateX = (y - centerY) / 20;
            const rotateY = (centerX - x) / 20;
            
            card.style.transform = `perspective(1000px) rotateX(${rotateX}deg) rotateY(${rotateY}deg) translateY(-8px)`;
        });
        
        card.addEventListener('mouseleave', () => {
            card.style.transform = '';
        });
    });

    // ===== Preload Critical Resources =====
    function preloadResources() {
        const links = [
            { rel: 'preconnect', href: 'https://fonts.googleapis.com' },
            { rel: 'preconnect', href: 'https://fonts.gstatic.com', crossorigin: true }
        ];
        
        links.forEach(linkData => {
            const link = document.createElement('link');
            Object.keys(linkData).forEach(key => {
                if (key === 'crossorigin') {
                    link.setAttribute(key, linkData[key]);
                } else {
                    link[key] = linkData[key];
                }
            });
            document.head.appendChild(link);
        });
    }
    
    preloadResources();

    // ===== Console Easter Egg =====
    console.log('%c⚡ TFT LEGAL', 'color: #00F5FF; font-size: 24px; font-weight: bold; text-shadow: 0 0 10px #00F5FF;');
    console.log('%cDigital Legal Solutions for Web3 Era', 'color: #A000FF; font-size: 14px;');
    console.log('%cInterested in our services? Contact us at hello@tftlegal.com', 'color: #92A5C2; font-size: 12px;');

})();
```

---

## 🎭 **favicon.svg**

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <defs>
    <linearGradient id="grad" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" style="stop-color:#00F5FF;stop-opacity:1" />
      <stop offset="100%" style="stop-color:#A000FF;stop-opacity:1" />
    </linearGradient>
    <filter id="glow">
      <feGaussianBlur stdDeviation="2" result="coloredBlur"/>
      <feMerge>
        <feMergeNode in="coloredBlur"/>
        <feMergeNode in="SourceGraphic"/>
      </feMerge>
    </filter>
  </defs>
  
  <rect width="100" height="100" fill="#050505"/>
  
  <text x="50" y="65" font-family="Orbitron, monospace" font-size="48" font-weight="900" 
        text-anchor="middle" fill="url(#grad)" filter="url(#glow)">TFT</text>
</svg>
```

---

## 🤖 **robots.txt**

```
User-agent: *
Allow: /

Sitemap: https://tftlegal.github.io/sitemap.xml
```

---

## 🗺️ **sitemap.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://tftlegal.github.io/</loc>
    <lastmod>2024-01-01</lastmod>
    <changefreq>monthly</changefreq>
    <priority>1.0</priority>
  </url>
</urlset>
```

---

## 📋 **README.md**

```markdown
# TFT Legal — Digital Law & Cyber Security

🌐 **Live Site:** [tftlegal.github.io](https://tftlegal.github.io)

## 🚀 О проекте

Статический сайт-визитка юридической компании TFT Legal, специализирующейся на цифровом праве, blockchain, Web3 и международных проектах.

### ✨ Особенности

- 🎨 Cyberpunk дизайн с неоновыми эффектами
- 📱 Полностью адаптивный layout
- ⚡ Быстрая загрузка (чистый HTML/CSS/JS)
- 🔍 SEO оптимизирован
- ♿ Доступность (WCAG 2.1)
- 🎭 Анимации и интерактивность

### 🛠️ Технологии

- HTML5
- CSS3 (Grid, Flexbox, Custom Properties)
- Vanilla JavaScript (ES6+)
- Google Fonts (Orbitron, Inter)

### 📦 Структура

```
/
├── index.html          # Главная страница
├── styles.css          # Все стили
├── script.js           # JavaScript
├── favicon.svg         # Иконка сайта
├── robots.txt          # Индексация
├── sitemap.xml         # Карта сайта
└── assets/             # Медиа ресурсы
    └── og-image.jpg    # OpenGraph изображение
```

### 🚀 Деплой

Сайт автоматически публикуется через GitHub Pages при push в main.

### 📧 Контакты

- **Email:** hello@tftlegal.com
- **Telegram:** @tftlegal
- **GitHub:** github.com/tftlegal

### 📄 Лицензия

© 2024 TFT Legal. All rights reserved.
```

---

## 📝 **.gitignore**

```
# OS
.DS_Store
Thumbs.db
desktop.ini

# Editors
.vscode/
.idea/
*.sublime-*

# Logs
*.log
npm-debug.log*

# Temp
*.tmp
*.temp
.cache/
```

---

## 🎯 **Инструкция по развертыванию**

### 1️⃣ Удалите старые файлы Hugo

```bash
cd tftlegal.github.io
rm -rf archetypes/ content/ data/ i18n/ layouts/ themes/ hugo.toml
```

### 2️⃣ Создайте новые файлы

Скопируйте все файлы выше в корень репозитория.

### 3️⃣ Создайте папку assets и добавьте og-image.jpg

```bash
mkdir assets
# Добавьте изображение для OpenGraph (рекомендуемый размер: 1200x630px)
```

### 4️⃣ Настройте GitHub Pages

1. Откройте Settings → Pages
2. Source: Deploy from a branch
3. Branch: `main` / `root`
4. Save

### 5️⃣ Залейте изменения

```bash
git add .
git commit -m "Redesign: Cyberpunk static site"
git push origin main
```

### ✅ Готово!

Сайт будет доступен по адресу: `https://tftlegal.github.io`

---

## 🎨 Кастомизация

### Изменить цвета

Отредактируйте CSS переменные в `styles.css`:

```css
:root {
    --cyan: #00F5FF;
    --purple: #A000FF;
    /* ... */
}
```

### Добавить контент

Редактируйте секции в `index.html`.

### Изменить шрифты

Замените ссылки на Google Fonts в `<head>`.

---

Теперь у вас есть полностью рабочий статический сайт в стиле киберпанк, готовый к публикации на GitHub Pages! 🚀

# GEMINI

Для того чтобы полностью перевести ваш сайт из шаблона Hugo в чистый, производительный и эффектный одностраничный сайт-визитку в стиле **Cyberpunk** (для размещения на GitHub Pages), мы полностью удалим всю структуру Hugo и заменим её на оптимизированный стек: **HTML5, CSS3 (с CSS-переменными и анимацией) и нативный JavaScript**.

Ниже представлена пошаговая инструкция по очистке проекта, а также готовый код каждого файла новой структуры.

---

### Шаг 1. Очистка репозитория

Удалите все старые файлы Hugo. В корневом каталоге вашего репозитория `tftlegal.github.io` выполните команду (или удалите вручную):

```bash
# Удалить все старые каталоги и файлы Hugo
rm -rf archetypes assets content data hugo.toml i18n layouts static themes
```

В результате у вас должна остаться пустая папка. Теперь создадим новую, чистую и легкую структуру:

```text
tftlegal.github.io/
├── index.html
├── styles.css
├── script.js
├── favicon.svg
└── assets/
    └── og-image.jpg
```

---

### Шаг 2. Исходный код файлов нового проекта

#### 1. Главный файл: `index.html`
Этот файл содержит семантическую разметку, SEO-оптимизацию (включая OpenGraph и мета-теги), подключение киберпанк-шрифтов (`Orbitron` для заголовков, `Inter` для текста) и всю структуру вашей визитки с интерактивным терминалом.

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>TFT Legal — Digital Law & Cyber Security</title>

    <meta name="description" content="Юридическое сопровождение цифрового бизнеса, Web3, Blockchain, IT и международных проектов.">
    <meta name="keywords" content="TFT Legal, Blockchain, Web3, Crypto, IT Law, Smart Contracts, Compliance, Юрист Блокчейн, Юрист IT">
    <meta name="author" content="TFT Legal">
    <meta name="theme-color" content="#050505">

    <!-- Open Graph / Facebook -->
    <meta property="og:type" content="website">
    <meta property="og:url" content="https://tftlegal.github.io/">
    <meta property="og:title" content="TFT Legal — Digital Law & Cyber Security">
    <meta property="og:description" content="Юридическое сопровождение цифрового бизнеса, Web3, Blockchain и IT проектов.">
    <meta property="og:image" content="assets/og-image.jpg">

    <!-- Twitter -->
    <meta property="twitter:card" content="summary_large_image">
    <meta property="twitter:url" content="https://tftlegal.github.io/">
    <meta property="twitter:title" content="TFT Legal — Digital Law & Cyber Security">
    <meta property="twitter:description" content="Юридическое сопровождение цифрового бизнеса, Web3, Blockchain и IT проектов.">
    <meta property="twitter:image" content="assets/og-image.jpg">

    <!-- Favicon -->
    <link rel="icon" type="image/svg+xml" href="favicon.svg">

    <!-- Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;700;900&family=Inter:wght@300;400;500;600&family=Fira+Code:wght@400;500&display=swap" rel="stylesheet">

    <!-- Styles -->
    <link rel="stylesheet" href="styles.css">
</head>
<body>

    <!-- Cyberpunk Background Effects -->
    <div class="grid-overlay"></div>
    <div class="scanlines"></div>
    <div class="noise-overlay"></div>

    <!-- Header -->
    <header>
        <div class="container header-container">
            <a class="logo" href="#hero">
                TFT<span>LEGAL</span>
            </a>
            
            <button class="menu-toggle" aria-label="Открыть меню" aria-expanded="false">
                <span></span>
                <span></span>
                <span></span>
            </button>

            <nav class="nav-menu">
                <a href="#about" class="nav-link">About</a>
                <a href="#services" class="nav-link">Services</a>
                <a href="#advantages" class="nav-link">Advantages</a>
                <a href="#contacts" class="nav-link">Contacts</a>
            </nav>
        </div>
    </header>

    <main>
        <!-- Hero Section -->
        <section id="hero" class="hero-section">
            <div class="container hero-grid">
                <div class="hero-left">
                    <div class="neon-badge animate-fade-in">
                        <span class="pulse-dot"></span> DIGITAL • BLOCKCHAIN • WEB3
                    </div>
                    
                    <h1 class="glitch-title" data-text="LEGAL FOR THE DIGITAL ERA">
                        LEGAL <br>
                        <span>FOR THE</span> <br>
                        DIGITAL ERA
                    </h1>
                    
                    <p class="hero-text">
                        TFT Legal предоставляет профессиональное юридическое сопровождение 
                        технологических компаний, криптопроектов, международного бизнеса 
                        и инновационных цифровых платформ.
                    </p>
                    
                    <div class="hero-buttons">
                        <a href="#contacts" class="btn btn-primary">
                            <span>CONTACT</span>
                        </a>
                        <a href="#services" class="btn btn-secondary">
                            <span>SERVICES</span>
                        </a>
                    </div>
                </div>
                
                <div class="hero-right">
                    <div class="terminal">
                        <div class="terminal-header">
                            <div class="terminal-buttons">
                                <span class="t-btn red"></span>
                                <span class="t-btn yellow"></span>
                                <span class="t-btn green"></span>
                            </div>
                            <span class="terminal-title">system_status.sh</span>
                        </div>
                        <div class="terminal-body">
                            <div id="terminal-output" class="terminal-code">
                                <!-- JavaScript заполнит это поле эффектом печати -->
                                <span class="cursor">_</span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <!-- About Section -->
        <section id="about" class="section-reveal">
            <div class="container">
                <h2 class="section-title" data-text="ABOUT">ABOUT</h2>
                <div class="glass-card about-card">
                    <div class="card-glow"></div>
                    <p class="accent-paragraph">
                        TFT Legal — высокоспециализированная команда экспертов, 
                        сопровождающая технологические и цифровые проекты на глобальном международном рынке.
                    </p>
                    <p>
                        Мы успешно консультируем и защищаем интересы бизнеса в сферах Blockchain, Web3, 
                        децентрализованных финансов (DeFi), криптоактивов, интеллектуальной собственности, 
                        информационных технологий (IT), международного корпоративного права и глобального комплаенса.
                    </p>
                </div>
            </div>
        </section>

        <!-- Services Section -->
        <section id="services" class="section-reveal">
            <div class="container">
                <h2 class="section-title" data-text="SERVICES">SERVICES</h2>
                <div class="services-grid">
                    
                    <article class="service-card">
                        <div class="card-border"></div>
                        <div class="card-content">
                            <div class="card-icon">⚡</div>
                            <h3>Blockchain</h3>
                            <p>Полное юридическое структурирование криптопроектов, токенизации активов, создания и сопровождения DAO, NFT-экосистем и выпуска цифровых финансовых активов.</p>
                        </div>
                    </article>

                    <article class="service-card">
                        <div class="card-border"></div>
                        <div class="card-content">
                            <div class="card-icon">💻</div>
                            <h3>IT Law</h3>
                            <p>Разработка технологических контрактов, лицензионных соглашений, SaaS договоров, договоров разработки ПО (Software Development), составление Privacy Policy и обеспечение соответствия GDPR.</p>
                        </div>
                    </article>

                    <article class="service-card">
                        <div class="card-border"></div>
                        <div class="card-content">
                            <div class="card-icon">🌐</div>
                            <h3>Corporate</h3>
                            <p>Создание холдинговых структур, регистрация IT-компаний в дружественных и классических юрисдикциях, сопровождение M&A сделок, инвестиционных раундов и венчурного финансирования.</p>
                        </div>
                    </article>

                    <article class="service-card">
                        <div class="card-border"></div>
                        <div class="card-content">
                            <div class="card-icon">🛡️</div>
                            <h3>Compliance</h3>
                            <p>Разработка политик AML/KYC, правовой аудит бизнес-моделей (Due Diligence), оценка юридических рисков и подготовка платформ к прохождению аудита регуляторами.</p>
                        </div>
                    </article>

                </div>
            </div>
        </section>

        <!-- Advantages Section -->
        <section id="advantages" class="section-reveal">
            <div class="container">
                <h2 class="section-title" data-text="ADVANTAGES">ADVANTAGES</h2>
                <div class="stats-grid">
                    
                    <div class="stat-item">
                        <div class="stat-number">10+</div>
                        <div class="stat-divider"></div>
                        <p class="stat-text">Years Experience</p>
                    </div>

                    <div class="stat-item">
                        <div class="stat-number">Global</div>
                        <div class="stat-divider"></div>
                        <p class="stat-text">International Projects</p>
                    </div>

                    <div class="stat-item">
                        <div class="stat-number">24/7</div>
                        <div class="stat-divider"></div>
                        <p class="stat-text">Support System</p>
                    </div>

                    <div class="stat-item">
                        <div class="stat-number">Web3</div>
                        <div class="stat-divider"></div>
                        <p class="stat-text">Digital Expertise</p>
                    </div>

                </div>
            </div>
        </section>

        <!-- Contacts Section -->
        <section id="contacts" class="section-reveal">
            <div class="container">
                <h2 class="section-title" data-text="CONTACT">CONTACT</h2>
                <div class="glass-card contact-card">
                    <div class="card-glow"></div>
                    <div class="contact-methods">
                        
                        <div class="contact-item">
                            <span class="contact-label">EMAIL:</span>
                            <a href="mailto:hello@tftlegal.com" class="contact-value">hello@tftlegal.com</a>
                        </div>
                        
                        <div class="contact-item">
                            <span class="contact-label">GITHUB:</span>
                            <a href="https://github.com/tftlegal" target="_blank" rel="noopener noreferrer" class="contact-value">github.com/tftlegal</a>
                        </div>
                        
                        <div class="contact-item">
                            <span class="contact-label">TELEGRAM:</span>
                            <a href="https://t.me/tftlegal" target="_blank" rel="noopener noreferrer" class="contact-value">@tftlegal</a>
                        </div>

                    </div>
                </div>
            </div>
        </section>
    </main>

    <!-- Footer -->
    <footer>
        <div class="container footer-container">
            <p class="copyright">© 2026 TFT Legal. All Rights Reserved.</p>
            <p class="footer-moto">DIGITAL LEGAL SOLUTIONS</p>
        </div>
    </footer>

    <!-- Scripts -->
    <script src="script.js"></script>
</body>
</html>
```

---

#### 2. Файл стилей: `styles.css`
Этот файл стилизует ваш сайт в современной эстетике Cyberpunk: сочетание темного матового фона с яркими неоновыми разделителями, эффектами свечения (`drop-shadow` и `box-shadow`), эффектом матового стекла (`backdrop-filter`) и анимациями сканирующих линий (CRT-эффект). 

```css
/* ==========================================================================
   TFT LEGAL — CYBERPUNK 2026 PRODUCTION CSS
   ========================================================================== */

:root {
    --bg-color: #030303;
    --card-bg: rgba(10, 10, 14, 0.75);
    
    /* Neon Colors */
    --neon-cyan: #00f5ff;
    --neon-purple: #9d00ff;
    --neon-pink: #ff007f;
    
    /* Texts */
    --text-primary: #e8f4ff;
    --text-muted: #8fa0ba;
    
    /* Borders & Outlines */
    --cyan-border: rgba(0, 245, 255, 0.2);
    --purple-border: rgba(157, 0, 255, 0.25);
    
    /* Shadows */
    --glow-cyan: 0 0 15px rgba(0, 245, 255, 0.35), 0 0 30px rgba(0, 245, 255, 0.15);
    --glow-purple: 0 0 15px rgba(157, 0, 255, 0.35), 0 0 30px rgba(157, 0, 255, 0.15);
    --glow-pink: 0 0 15px rgba(255, 0, 127, 0.4), 0 0 30px rgba(255, 0, 127, 0.15);

    --transition-fast: 0.25s ease;
    --transition-smooth: 0.4s cubic-bezier(0.16, 1, 0.3, 1);
}

/* ==========================================================================
   BASE STYLES & RESET
   ========================================================================== */

* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

html {
    scroll-behavior: smooth;
    scrollbar-width: thin;
    scrollbar-color: var(--neon-cyan) var(--bg-color);
}

body {
    background: var(--bg-color);
    color: var(--text-primary);
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
    overflow-x: hidden;
    line-height: 1.625;
}

/* Custom Scrollbar */
::-webkit-scrollbar {
    width: 6px;
}
::-webkit-scrollbar-track {
    background: var(--bg-color);
}
::-webkit-scrollbar-thumb {
    background: var(--neon-purple);
    border-radius: 3px;
    border: 1px solid var(--neon-cyan);
}

/* ==========================================================================
   CYBERPUNK CANVAS EFFECTS (BACKGROUNDS)
   ========================================================================== */

/* Neon Gradients Overlay */
body::before {
    content: "";
    position: fixed;
    inset: 0;
    background: 
        radial-gradient(circle at 10% 20%, rgba(0, 245, 255, 0.08) 0%, transparent 45%),
        radial-gradient(circle at 90% 15%, rgba(157, 0, 255, 0.12) 0%, transparent 50%),
        radial-gradient(circle at 50% 85%, rgba(255, 0, 127, 0.06) 0%, transparent 40%);
    pointer-events: none;
    z-index: -5;
}

/* Technical HUD Grid */
.grid-overlay {
    position: fixed;
    inset: 0;
    background-image: 
        linear-gradient(rgba(0, 245, 255, 0.04) 1px, transparent 1px),
        linear-gradient(90deg, rgba(0, 245, 255, 0.04) 1px, transparent 1px);
    background-size: 60px 60px;
    opacity: 0.6;
    pointer-events: none;
    z-index: -4;
    mask-image: radial-gradient(circle, black 40%, transparent 95%);
}

/* Scanlines (CRT Simulation) */
.scanlines {
    position: fixed;
    inset: 0;
    pointer-events: none;
    background: repeating-linear-gradient(
        0deg,
        rgba(255, 255, 255, 0.015),
        rgba(255, 255, 255, 0.015) 1px,
        transparent 2px,
        transparent 4px
    );
    z-index: -2;
}

/* Analog CRT Flickering/Noise overlay */
.noise-overlay {
    position: fixed;
    inset: 0;
    pointer-events: none;
    background: url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noiseFilter'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noiseFilter)'/%3E%3C/svg%3E");
    opacity: 0.02;
    z-index: -3;
}

/* ==========================================================================
   TYPOGRAPHY & GLOBAL LAYOUT
   ========================================================================== */

.container {
    width: min(1200px, 90%);
    margin: 0 auto;
}

h1, h2, h3, h4 {
    font-family: 'Orbitron', sans-serif;
    text-transform: uppercase;
    letter-spacing: 2px;
}

.section-title {
    font-size: clamp(2rem, 4vw, 3rem);
    color: var(--neon-cyan);
    margin-bottom: 3.5rem;
    position: relative;
    display: inline-block;
    text-shadow: var(--glow-cyan);
    font-weight: 800;
}

.section-title::after {
    content: "";
    position: absolute;
    bottom: -8px;
    left: 0;
    width: 60%;
    height: 3px;
    background: linear-gradient(90deg, var(--neon-cyan), transparent);
}

section {
    padding: clamp(80px, 10vw, 150px) 0;
}

/* ==========================================================================
   HEADER & NAVIGATION
   ========================================================================== */

header {
    position: fixed;
    width: 100%;
    top: 0;
    z-index: 999;
    background: rgba(3, 3, 3, 0.7);
    border-bottom: 1px solid rgba(0, 245, 255, 0.15);
    backdrop-filter: blur(12px);
    -webkit-backdrop-filter: blur(12px);
    transition: var(--transition-smooth);
}

.header-container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 85px;
}

.logo {
    text-decoration: none;
    color: #fff;
    font-size: 1.6rem;
    font-family: 'Orbitron', sans-serif;
    font-weight: 900;
    letter-spacing: 3px;
    transition: var(--transition-fast);
}

.logo span {
    color: var(--neon-cyan);
    text-shadow: var(--glow-cyan);
}

.logo:hover {
    transform: skew(-5deg);
}

/* Navigation Links */
.nav-menu {
    display: flex;
    gap: 40px;
}

.nav-link {
    font-family: 'Orbitron', sans-serif;
    font-size: 0.9rem;
    font-weight: 500;
    color: var(--text-muted);
    text-decoration: none;
    text-transform: uppercase;
    transition: var(--transition-fast);
    position: relative;
    padding: 6px 0;
    letter-spacing: 1px;
}

.nav-link:hover {
    color: var(--neon-cyan);
    text-shadow: var(--glow-cyan);
}

.nav-link::after {
    content: "";
    position: absolute;
    bottom: 0;
    left: 0;
    width: 0;
    height: 2px;
    background: var(--neon-cyan);
    box-shadow: var(--glow-cyan);
    transition: var(--transition-fast);
}

.nav-link:hover::after {
    width: 100%;
}

/* Burger Menu Toggle */
.menu-toggle {
    display: none;
    flex-direction: column;
    gap: 6px;
    background: transparent;
    border: none;
    cursor: pointer;
    z-index: 1000;
}

.menu-toggle span {
    display: block;
    width: 28px;
    height: 2px;
    background: var(--neon-cyan);
    box-shadow: var(--glow-cyan);
    transition: var(--transition-fast);
}

/* ==========================================================================
   HERO SECTION
   ========================================================================== */

.hero-section {
    min-height: 100vh;
    display: flex;
    align-items: center;
    padding-top: 120px;
    position: relative;
}

.hero-grid {
    display: grid;
    grid-template-columns: 1.25fr 1fr;
    gap: 60px;
    align-items: center;
}

.neon-badge {
    display: inline-flex;
    align-items: center;
    gap: 10px;
    border: 1px solid var(--neon-cyan);
    color: var(--neon-cyan);
    background: rgba(0, 245, 255, 0.05);
    padding: 8px 20px;
    border-radius: 50px;
    font-size: 0.8rem;
    font-family: 'Orbitron', sans-serif;
    font-weight: 600;
    letter-spacing: 2px;
    margin-bottom: 2rem;
    box-shadow: var(--glow-cyan);
    width: fit-content;
}

.pulse-dot {
    width: 8px;
    height: 8px;
    background-color: var(--neon-cyan);
    border-radius: 50%;
    box-shadow: var(--glow-cyan);
    animation: badgePulse 2s infinite;
}

@keyframes badgePulse {
    0% { transform: scale(0.9); opacity: 0.5; }
    50% { transform: scale(1.2); opacity: 1; }
    100% { transform: scale(0.9); opacity: 0.5; }
}

.glitch-title {
    font-size: clamp(2.5rem, 5.5vw, 4.8rem);
    font-weight: 900;
    line-height: 1.05;
    margin-bottom: 1.5rem;
    position: relative;
}

.glitch-title span {
    color: var(--neon-purple);
    text-shadow: var(--glow-purple);
}

.hero-text {
    font-size: clamp(1rem, 1.25vw, 1.25rem);
    color: var(--text-muted);
    max-width: 620px;
    margin-bottom: 2.5rem;
}

/* Buttons System */
.hero-buttons {
    display: flex;
    gap: 20px;
}

.btn {
    text-decoration: none;
    font-family: 'Orbitron', sans-serif;
    font-size: 0.95rem;
    font-weight: 700;
    padding: 16px 36px;
    border-radius: 4px;
    transition: var(--transition-smooth);
    position: relative;
    letter-spacing: 2px;
    overflow: hidden;
}

.btn-primary {
    background: var(--neon-cyan);
    color: #000;
    box-shadow: var(--glow-cyan);
    border: none;
}

.btn-primary:hover {
    transform: translateY(-3px);
    box-shadow: 0 0 25px var(--neon-cyan), 0 0 45px rgba(0, 245, 255, 0.4);
}

.btn-secondary {
    background: transparent;
    color: var(--neon-purple);
    border: 1px solid var(--neon-purple);
    box-shadow: inset 0 0 10px rgba(157, 0, 255, 0.1);
}

.btn-secondary:hover {
    background: var(--neon-purple);
    color: #fff;
    transform: translateY(-3px);
    box-shadow: var(--glow-purple);
}

/* Interactive Terminal Panel */
.terminal {
    background: rgba(10, 10, 15, 0.9);
    border: 1px solid var(--purple-border);
    border-radius: 8px;
    box-shadow: var(--glow-purple);
    overflow: hidden;
    position: relative;
}

.terminal::before {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: linear-gradient(180deg, transparent 96%, rgba(157, 0, 255, 0.08) 96%),
                linear-gradient(90deg, transparent 96%, rgba(157, 0, 255, 0.08) 96%);
    background-size: 20px 20px;
    pointer-events: none;
}

.terminal-header {
    background: #0f0f16;
    padding: 14px 20px;
    display: flex;
    align-items: center;
    position: relative;
    border-bottom: 1px solid rgba(157, 0, 255, 0.15);
}

.terminal-buttons {
    display: flex;
    gap: 8px;
}

.t-btn {
    width: 10px;
    height: 10px;
    border-radius: 50%;
}

.t-btn.red { background-color: var(--neon-pink); box-shadow: 0 0 6px var(--neon-pink); }
.t-btn.yellow { background-color: #ffd000; box-shadow: 0 0 6px #ffd000; }
.t-btn.green { background-color: var(--neon-cyan); box-shadow: 0 0 6px var(--neon-cyan); }

.terminal-title {
    margin: 0 auto;
    font-family: 'Fira Code', monospace;
    font-size: 0.8rem;
    color: var(--text-muted);
}

.terminal-body {
    padding: 24px;
    min-height: 280px;
    display: flex;
    flex-direction: column;
}

.terminal-code {
    font-family: 'Fira Code', 'Courier New', monospace;
    font-size: 0.88rem;
    color: var(--text-primary);
    line-height: 1.8;
}

.terminal-line {
    margin-bottom: 6px;
    display: block;
}

.term-tag { color: var(--neon-cyan); font-weight: 500; }
.term-val { color: var(--neon-pink); }
.term-ok { color: #00ff66; font-weight: bold; }

.cursor {
    animation: blink 0.9s steps(2, start) infinite;
    color: var(--neon-cyan);
    font-weight: bold;
}

@keyframes blink {
    to { visibility: hidden; }
}

/* ==========================================================================
   ABOUT SECTION & GLASSMORPHISM
   ========================================================================== */

.glass-card {
    background: var(--card-bg);
    border: 1px solid var(--cyan-border);
    border-radius: 12px;
    padding: clamp(30px, 5vw, 60px);
    position: relative;
    overflow: hidden;
    box-shadow: 0 20px 40px rgba(0,0,0,0.5);
    backdrop-filter: blur(14px);
    -webkit-backdrop-filter: blur(14px);
}

.about-card {
    border-left: 4px solid var(--neon-cyan);
}

.card-glow {
    position: absolute;
    width: 250px;
    height: 250px;
    background: radial-gradient(circle, rgba(0, 245, 255, 0.08) 0%, transparent 70%);
    top: -125px;
    right: -125px;
    pointer-events: none;
}

.accent-paragraph {
    font-size: 1.4rem;
    line-height: 1.5;
    font-weight: 500;
    color: #fff;
    margin-bottom: 2rem;
}

.glass-card p:not(.accent-paragraph) {
    color: var(--text-muted);
    font-size: 1.05rem;
    max-width: 950px;
}

/* ==========================================================================
   SERVICES GRID & CARDS
   ========================================================================== */

.services-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
    gap: 30px;
}

.service-card {
    position: relative;
    background: var(--card-bg);
    border: 1px solid var(--purple-border);
    border-radius: 8px;
    overflow: hidden;
    transition: var(--transition-smooth);
    height: 100%;
}

.service-card:hover {
    transform: translateY(-8px);
    border-color: var(--neon-cyan);
    box-shadow: var(--glow-cyan);
}

.card-content {
    padding: 40px 30px;
    z-index: 2;
    position: relative;
}

.card-icon {
    font-size: 2.2rem;
    margin-bottom: 1.5rem;
    filter: drop-shadow(0 0 8px rgba(157, 0, 255, 0.4));
    transition: var(--transition-fast);
}

.service-card:hover .card-icon {
    filter: drop-shadow(0 0 12px var(--neon-cyan));
    transform: scale(1.1);
}

.service-card h3 {
    font-size: 1.35rem;
    color: #fff;
    margin-bottom: 1rem;
    font-weight: 700;
}

.service-card p {
    font-size: 0.95rem;
    color: var(--text-muted);
}

/* ==========================================================================
   ADVANTAGES SECTION (STATS)
   ========================================================================== */

.stats-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
    gap: 30px;
}

.stat-item {
    background: rgba(10, 10, 15, 0.5);
    border: 1px solid rgba(255, 255, 255, 0.05);
    padding: 40px 30px;
    text-align: center;
    border-radius: 8px;
    position: relative;
    transition: var(--transition-smooth);
}

.stat-item:hover {
    border-color: var(--neon-pink);
    box-shadow: var(--glow-pink);
    transform: translateY(-5px);
}

.stat-number {
    font-family: 'Orbitron', sans-serif;
    font-size: 3.5rem;
    font-weight: 900;
    color: #fff;
    line-height: 1;
    margin-bottom: 1rem;
    background: linear-gradient(135deg, #fff, var(--neon-pink));
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    filter: drop-shadow(0 0 10px rgba(255, 0, 127, 0.3));
}

.stat-divider {
    width: 50px;
    height: 2px;
    background: var(--neon-pink);
    margin: 0 auto 1.25rem;
    box-shadow: var(--glow-pink);
}

.stat-text {
    font-size: 0.95rem;
    text-transform: uppercase;
    letter-spacing: 2px;
    font-family: 'Orbitron', sans-serif;
    color: var(--text-muted);
}

/* ==========================================================================
   CONTACT SECTION
   ========================================================================== */

.contact-card {
    border: 1px solid var(--purple-border);
}

.contact-card::before {
    content: "";
    position: absolute;
    left: 0;
    top: 0;
    width: 4px;
    height: 100%;
    background-color: var(--neon-purple);
    box-shadow: var(--glow-purple);
}

.contact-methods {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 40px;
}

.contact-item {
    display: flex;
    flex-direction: column;
    gap: 10px;
}

.contact-label {
    font-family: 'Orbitron', sans-serif;
    font-size: 0.85rem;
    letter-spacing: 2px;
    color: var(--neon-purple);
    text-shadow: var(--glow-purple);
    font-weight: 600;
}

.contact-value {
    font-size: clamp(1.1rem, 1.6vw, 1.5rem);
    font-weight: 500;
    color: #fff;
    text-decoration: none;
    transition: var(--transition-fast);
    word-break: break-all;
}

.contact-value:hover {
    color: var(--neon-cyan);
    text-shadow: var(--glow-cyan);
    transform: translateX(4px);
}

/* ==========================================================================
   FOOTER
   ========================================================================== */

footer {
    background: #020202;
    border-top: 1px solid rgba(255, 255, 255, 0.05);
    padding: 40px 0;
}

.footer-container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 20px;
}

.copyright {
    color: var(--text-muted);
    font-size: 0.9rem;
}

.footer-moto {
    font-family: 'Orbitron', sans-serif;
    font-size: 0.85rem;
    font-weight: 700;
    letter-spacing: 3px;
    color: var(--neon-cyan);
    text-shadow: var(--glow-cyan);
}

/* ==========================================================================
   RESPONSIVE DESIGN (MEDIA QUERIES)
   ========================================================================== */

@media (max-width: 992px) {
    .hero-grid {
        grid-template-columns: 1fr;
        gap: 50px;
    }
    
    .hero-left {
        text-align: center;
        display: flex;
        flex-direction: column;
        align-items: center;
    }
    
    .hero-buttons {
        justify-content: center;
    }
    
    .terminal {
        max-width: 600px;
        margin: 0 auto;
        width: 100%;
    }
}

@media (max-width: 768px) {
    .menu-toggle {
        display: flex;
    }
    
    .nav-menu {
        position: fixed;
        top: 85px;
        left: -100%;
        width: 100%;
        height: calc(100vh - 85px);
        background: rgba(3, 3, 5, 0.98);
        border-top: 1px solid rgba(0, 245, 255, 0.15);
        backdrop-filter: blur(20px);
        flex-direction: column;
        align-items: center;
        justify-content: center;
        gap: 35px;
        transition: var(--transition-smooth);
        z-index: 998;
    }
    
    .nav-menu.active {
        left: 0;
    }
    
    .header-container {
        height: 75px;
    }
    
    .nav-link {
        font-size: 1.2rem;
    }

    /* Burger animation when menu is active */
    .menu-toggle.active span:nth-child(1) {
        transform: translateY(8px) rotate(45deg);
    }
    .menu-toggle.active span:nth-child(2) {
        opacity: 0;
    }
    .menu-toggle.active span:nth-child(3) {
        transform: translateY(-8px) rotate(-45deg);
    }
}

/* Scroll Animation classes (handled by Intersection Observer) */
.section-reveal {
    opacity: 0;
    transform: translateY(40px);
    transition: opacity 0.8s ease, transform 0.8s ease;
}

.section-reveal.active {
    opacity: 1;
    transform: translateY(0);
}
```

---

#### 3. Интерактивный JS: `script.js`
Этот файл добавляет интерактивность: симулирует настоящую терминальную печать системного статуса при загрузке, реализует анимацию плавного появления блоков при скролле (Intersection Observer) и обрабатывает мобильную навигацию ("бургер"-меню).

```javascript
/* ==========================================================================
   TFT LEGAL — INTERACTIVE SCRIPT
   ========================================================================== */

document.addEventListener("DOMContentLoaded", () => {
    
    // --- 1. ТЕРМИНАЛЬНАЯ ИНТЕРАКТИВНАЯ АНИМАЦИЯ ПЕЧАТИ ---
    const terminalLines = [
        { text: "> INITIALIZING SECURE JURISDICTION INTERFACE...", delay: 200, isTag: true },
        { text: "Connection ........ ", tag: "ONLINE", valClass: "term-ok", delay: 600 },
        { text: "Jurisdiction ...... ", tag: "GLOBAL", valClass: "term-val", delay: 400 },
        { text: "Compliance ........ ", tag: "VERIFIED", valClass: "term-ok", delay: 500 },
        { text: "Blockchain ........ ", tag: "ACTIVE_NODE", valClass: "term-val", delay: 400 },
        { text: "Smart Contracts ... ", tag: "SECURED & AUDITED", valClass: "term-ok", delay: 600 },
        { text: "Digital Assets .... ", tag: "PROTECTED BY LAW", valClass: "term-val", delay: 300 },
        { text: "> SECURE SESSION CONNECTED SUCCESSFULLY.", delay: 300, isTag: true }
    ];

    const terminalContainer = document.getElementById("terminal-output");
    
    if (terminalContainer) {
        let currentLine = 0;
        terminalContainer.innerHTML = ""; // Очищаем контейнер

        function writeLine() {
            if (currentLine >= terminalLines.length) {
                // Добавляем финальный мигающий курсор
                const cursorSpan = document.createElement("span");
                cursorSpan.className = "cursor";
                cursorSpan.innerHTML = "_";
                terminalContainer.appendChild(cursorSpan);
                return;
            }

            const lineData = terminalLines[currentLine];
            const lineElement = document.createElement("span");
            lineElement.className = "terminal-line";
            
            if (lineData.isTag) {
                lineElement.className += " term-tag";
                lineElement.textContent = lineData.text;
                terminalContainer.appendChild(lineElement);
                currentLine++;
                setTimeout(writeLine, lineData.delay);
            } else {
                // Имитируем постепенную печать
                lineElement.textContent = lineData.text;
                terminalContainer.appendChild(lineElement);
                
                let i = 0;
                lineElement.textContent = "";
                
                function typeChar() {
                    if (i < lineData.text.length) {
                        lineElement.textContent += lineData.text.charAt(i);
                        i++;
                        setTimeout(typeChar, 15);
                    } else {
                        // Печатаем цветной статус после завершения строки
                        const tagSpan = document.createElement("span");
                        tagSpan.className = lineData.valClass;
                        tagSpan.textContent = lineData.tag;
                        lineElement.appendChild(tagSpan);
                        
                        currentLine++;
                        setTimeout(writeLine, lineData.delay);
                    }
                }
                typeChar();
            }
        }
        
        // Запуск терминала с небольшой задержкой
        setTimeout(writeLine, 800);
    }

    // --- 2. МОБИЛЬНОЕ БУРГЕР-МЕНЮ ---
    const menuToggle = document.querySelector(".menu-toggle");
    const navMenu = document.querySelector(".nav-menu");
    const navLinks = document.querySelectorAll(".nav-link");

    if (menuToggle && navMenu) {
        menuToggle.addEventListener("click", () => {
            const isActive = menuToggle.classList.toggle("active");
            navMenu.classList.toggle("active");
            menuToggle.setAttribute("aria-expanded", isActive);
        });

        // Закрытие меню при клике на любой из пунктов навигации
        navLinks.forEach(link => {
            link.addEventListener("click", () => {
                menuToggle.classList.remove("active");
                navMenu.classList.remove("active");
                menuToggle.setAttribute("aria-expanded", "false");
            });
        });
    }

    // --- 3. АНИМАЦИЯ ПОЯВЛЕНИЯ БЛОКОВ ПРИ СКРОЛЛЕ (INTERSECTION OBSERVER) ---
    const revealSections = document.querySelectorAll(".section-reveal");
    
    if (revealSections.length > 0) {
        const revealCallback = (entries, observer) => {
            entries.forEach(entry => {
                if (entry.isIntersecting) {
                    entry.target.classList.add("active");
                    // Прекращаем наблюдение за элементом после того, как он показан
                    observer.unobserve(entry.target);
                }
            });
        };

        const revealObserver = new IntersectionObserver(revealCallback, {
            root: null, // Относительно вьюпорта
            threshold: 0.15, // Блок начинает открываться, если виден на 15%
            rootMargin: "0px"
        });

        revealSections.forEach(section => {
            revealObserver.observe(section);
        });
    }

    // --- 4. ИНТЕРАКТИВНЫЙ ЭФФЕКТ СВЕЧЕНИЯ ДЛЯ КАРТОЧЕК ---
    // Движение градиента за курсором мыши (эффект наведения на карточках)
    const glassCards = document.querySelectorAll(".glass-card, .service-card");
    glassCards.forEach(card => {
        card.addEventListener("mousemove", (e) => {
            const rect = card.getBoundingClientRect();
            const x = e.clientX - rect.left;
            const y = e.clientY - rect.top;
            
            // Если внутри карточки есть элемент glow, двигаем его за мышкой
            const glow = card.querySelector(".card-glow");
            if (glow) {
                glow.style.top = `${y - 125}px`;
                glow.style.left = `${x - 125}px`;
            }
        });
    });
});
```

---

#### 4. Иконка сайта: `favicon.svg`
Это современная векторная иконка в формате SVG. Скопируйте следующий код и создайте файл `favicon.svg` в корневом каталоге проекта.

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <defs>
    <linearGradient id="neonGlow" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" stop-color="#00f5ff" />
      <stop offset="100%" stop-color="#9d00ff" />
    </linearGradient>
    <filter id="glow">
      <feGaussianBlur stdDeviation="3" result="blur" />
      <feComposite in="SourceGraphic" in2="blur" operator="over" />
    </filter>
  </defs>
  <!-- Background -->
  <rect width="100" height="100" rx="18" fill="#050505" stroke="#00f5ff" stroke-width="2" />
  <!-- Logo Shapes -->
  <g filter="url(#glow)">
    <path d="M 30,30 L 70,30 L 50,55 Z" fill="none" stroke="url(#neonGlow)" stroke-width="5" stroke-linejoin="round"/>
    <path d="M 40,65 L 60,65" stroke="#ff007f" stroke-width="6" stroke-linecap="round" />
  </g>
</svg>
```

---

#### 5. Изображение для предпросмотра соцсетей: `assets/og-image.jpg`
Положите любое тематическое изображение размером **1200x630px** по пути `assets/og-image.jpg`. Это изображение будет выводиться при отправке ссылки в Telegram, WhatsApp, LinkedIn и другие платформы.

---

### Шаг 3. Деплой на GitHub Pages

Когда все файлы созданы в корневом каталоге вашего репозитория `tftlegal.github.io`, отправьте изменения на GitHub:

```bash
git add .
git commit -m "feat: complete rewrite to static cyberpunk landing"
git push origin main
```

Через 1-2 минуты ваш статический сайт на GitHub Pages обновится автоматически и будет доступен по адресу `https://tftlegal.github.io/`.

### Что изменилось в новой версии:
1. **Никакого оверхеда:** Больше нет Hugo, темы Archie и генерируемого тяжелого JS. Страница весит считанные килобайты и загружается мгновенно.
2. **Впечатляющий UX/UI:** Анимированный терминал на чистом JS создает сильное технологическое впечатление, соответствующее деятельности в сферах *Web3* и *Cyber Security*.
3. **Оптимизированный адаптив:** Шаблон идеально подстраивается как под огромные UHD экраны, так и под экраны мобильных устройств.
4. **Сохранение SEO:** Все мета-теги и оригинальные тексты о деятельности компании и ее преимуществах сохранены на русском языке.
