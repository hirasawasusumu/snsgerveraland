function showPanel(panelName) {
    const content = document.getElementById('content');
    switch(panelName) {
        case 'home':
            content.innerHTML = "<p>ホームパネルが表示されました。</p>";
            break;
        case 'news':
            content.innerHTML = "<p>ニュースパネルが表示されました。</p>";
            break;
        case 'profile':
            content.innerHTML = "<p>プロフィールパネルが表示されました。</p>";
            break;
        default:
            content.innerHTML = "<p>ここにコンテンツが表示されます。</p>";
    }
}
# snsgerveraland
SNSLAND
