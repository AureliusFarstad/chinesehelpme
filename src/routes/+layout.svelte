<div class="header">
    <div id="centered">
        <h1><a href="/">chinesehelp.me</a></h1>
        <nav>
            <ul>
              <li><a href="/">HSK Sniper 🔫</a></li>
              <li><a href="/read_a_text">Long Text Processer</a></li>
              <!-- <li><a href="/word_tiger">Text Processer Penguin 🐧</a></li> -->
            </ul>
        </nav>
    </div>
</div>

<div class="displacement"></div>

<slot></slot>

<style>
    .header {
        position: fixed;
        z-index: 10;
        top: 0px;
        left: 0px;

        width: 100%;
        height: 50px;
    
        background-image: linear-gradient(45deg, #CC0000, #FF9999);
    }

    .displacement {
        padding-top: 50px;
    }

    #centered {
        display: flex;

        margin: 0 auto;
        max-width: 1020px;
        width: 100%;
        height: 50px;
    }
    
    h1 {
        margin: 0px 10px 8px 10px;

        font-size: 24px;
        line-height: 50px;
        font-family: 'Futura', sans-serif;
        font-weight: 400;
        color: #ffffff;
    }

    nav {
        width: 100%;
    }

    ul {
        display: flex;
        flex-direction: row;

        list-style: none;
    }

    li {
        margin-right: 50px;

        font-family: 'Futura', sans-serif;
        font-weight: 400;
        color: #ffffff;
    }

    a {
        color: #ffffff;

        text-decoration: none;
    }
</style>