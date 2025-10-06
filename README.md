import React, { useState, useEffect } from "react";
最低な世の中を一発で変える力
最大ルーラ力
// Twitter-like single-file React app
// Usage: paste into a Vite/CRA project (src/App.jsx) with Tailwind CSS set up.
// Features:
// - Compose tweets, like, reply, retweet (local only)
// - Simple user switch (choose a display name)
// - Persist to localStorage
// - Search, simple profile view, timeline
// - Mobile-friendly layout using Tailwind

const SAMPLE_TWEETS = [
  {
    id: 1,
    user: "にゃんこ",
    handle: "@nyanko",
    content: "はじめてのつぶやき！",
    time: Date.now() - 1000 * 60 * 60,
    likes: 2,
    retweets: 1,
    replies: [],
  },
  {
    id: 2,
    user: "ふみかちゃん",
    handle: "@fumika",
    content: "デザイン仕事は楽しい〜✨",
    time: Date.now() - 1000 * 60 * 30,
    likes: 5,
    retweets: 2,
    replies: [],
  },
];

function timeAgo(ts) {
  const s = Math.floor((Date.now() - ts) / 1000);
  if (s < 60) return `${s}s`;
  const m = Math.floor(s / 60);
  if (m < 60) return `${m}m`;
  const h = Math.floor(m / 60);
  if (h < 24) return `${h}h`;
  const d = Math.floor(h / 24);
  return `${d}d`;
}

export default function App() {
  const [user, setUser] = useState(() => {
    return localStorage.getItem("tl_user") || "ゲスト";
  });
  const [text, setText] = useState("");
  const [tweets, setTweets] = useState(() => {
    try {
      const raw = localStorage.getItem("tl_tweets");
      return raw ? JSON.parse(raw) : SAMPLE_TWEETS;
    } catch (e) {
      return SAMPLE_TWEETS;
    }
  });
  const [query, setQuery] = useState("");
  const [view, setView] = useState("home"); // home | profile | search

  useEffect(() => {
    localStorage.setItem("tl_tweets", JSON.stringify(tweets));
  }, [tweets]);

  useEffect(() => {
    localStorage.setItem("tl_user", user);
  }, [user]);

  function postTweet() {
    if (!text.trim()) return;
    const newTweet = {
      id: Date.now(),
      user: user,
      handle: `@${user.replace(/\s+/g, "")}`,
      content: text.trim(),
      time: Date.now(),
      likes: 0,
      retweets: 0,
      replies: [],
    };
    setTweets([newTweet, ...tweets]);
    setText("");
    setView("home");
  }

  function toggleLike(id) {
    setTweets((t) => t.map((tw) => (tw.id === id ? { ...tw, likes: tw._liked ? tw.likes - 1 : tw.likes + 1, _liked: !tw._liked } : tw)));
  }

  function doRetweet(id) {
    setTweets((t) => t.map((tw) => (tw.id === id ? { ...tw, retweets: tw.retweets + 1 } : tw)));
  }

  function replyTo(id, replyText) {
    if (!replyText.trim()) return;
    setTweets((t) =>
      t.map((tw) => (tw.id === id ? { ...tw, replies: [{ user, content: replyText.trim(), time: Date.now() }, ...tw.replies] } : tw))
    );
  }

  function deleteTweet(id) {
    setTweets((t) => t.filter((tw) => tw.id !== id));
  }

  const filtered = tweets.filter((tw) => tw.content.includes(query) || tw.user.includes(query) || tw.handle.includes(query));

  return (
    <div className="min-h-screen bg-slate-50 text-slate-900">
      <div className="max-w-4xl mx-auto p-4 grid grid-cols-1 md:grid-cols-4 gap-4">
        {/* Left: profile / settings */}
        <aside className="md:col-span-1 bg-white p-4 rounded-2xl shadow-sm">
          <div className="flex items-center gap-3">
            <div className="w-14 h-14 rounded-full bg-gradient-to-br from-indigo-400 to-pink-400 flex items-center justify-center text-white font-bold">{user[0] || 'G'}</div>
            <div>
              <div className="font-semibold">{user}</div>
              <div className="text-sm text-slate-500">{`@${user.replace(/\s+/g, "").toLowerCase()}`}</div>
            </div>
          </div>

          <div className="mt-4">
            <label className="block text-sm text-slate-600">表示名を変更</label>
            <input
              value={user}
              onChange={(e) => setUser(e.target.value)}
              className="mt-2 w-full border rounded-md px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-slate-200"
            />
          </div>

          <div className="mt-4 flex flex-col gap-2">
            <button onClick={() => setView("home")} className={`py-2 rounded-lg text-sm ${view === "home" ? "bg-slate-100" : "bg-white"}`}>
              ホーム
            </button>
            <button onClick={() => setView("profile")} className={`py-2 rounded-lg text-sm ${view === "profile" ? "bg-slate-100" : "bg-white"}`}>
              プロフィール
            </button>
            <button onClick={() => setView("search")} className={`py-2 rounded-lg text-sm ${view === "search" ? "bg-slate-100" : "bg-white"}`}>
              検索
            </button>
          </div>
        </aside>

        {/* Middle: timeline */}
        <main className="md:col-span-2">
          <div className="bg-white p-4 rounded-2xl shadow-sm mb-4">
            <div className="flex items-start gap-3">
              <div className="w-12 h-12 rounded-full bg-indigo-300 flex items-center justify-center text-white font-bold">{user[0] || 'G'}</div>
              <div className="flex-1">
                <textarea
                  rows={3}
                  value={text}
                  onChange={(e) => setText(e.target.value)}
                  placeholder="今どうしてる？"
                  className="w-full border rounded-md p-3 text-sm resize-none focus:outline-none focus:ring-2 focus:ring-slate-200"
                />
                <div className="flex items-center justify-between mt-2">
                  <div className="text-sm text-slate-500">{text.length}/280</div>
                  <div className="flex items-center gap-2">
                    <button onClick={postTweet} className="px-4 py-2 bg-blue-500 text-white rounded-full text-sm">投稿</button>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div>
            {view === "home" && (
              <section>
                {tweets.length === 0 && <div className="p-6 text-center text-slate-500">まだ投稿がありません</div>}
                {tweets.map((tw) => (
                  <article key={tw.id} className="bg-white p-4 rounded-2xl mb-3 shadow-sm">
                    <div className="flex gap-3">
                      <div className="w-12 h-12 rounded-full bg-indigo-200 flex items-center justify-center font-bold text-white">{tw.user[0]}</div>
                      <div className="flex-1">
                        <div className="flex items-center justify-between">
                          <div>
                            <div className="font-semibold">{tw.user}</div>
                            <div className="text-xs text-slate-500">{tw.handle} · {timeAgo(tw.time)}</div>
                          </div>
                          <div className="text-xs text-slate-400">ID:{tw.id}</div>
                        </div>
                        <div className="mt-3 whitespace-pre-wrap">{tw.content}</div>

                        <div className="mt-3 flex items-center gap-4 text-sm text-slate-600">
                          <button onClick={() => toggleLike(tw.id)} className="flex items-center gap-1">
                            ❤️ {tw.likes}
                          </button>
                          <button onClick={() => doRetweet(tw.id)} className="flex items-center gap-1">🔁 {tw.retweets}</button>
                          <ReplyBox tweet={tw} onReply={(txt) => replyTo(tw.id, txt)} />
                          {tw.user === user && (
                            <button onClick={() => deleteTweet(tw.id)} className="text-red-500">削除</button>
                          )}
                        </div>

                        {tw.replies && tw.replies.length > 0 && (
                          <div className="mt-3 border-l pl-3 text-sm text-slate-700">
                            {tw.replies.map((r, i) => (
                              <div key={i} className="mb-2">
                                <div className="font-medium text-xs">{r.user} · {timeAgo(r.time)}</div>
                                <div className="text-sm">{r.content}</div>
                              </div>
                            ))}
                          </div>
                        )}
                      </div>
                    </div>
                  </article>
                ))}
              </section>
            )}

            {view === "profile" && (
              <section className="bg-white p-4 rounded-2xl shadow-sm">
                <h2 className="text-xl font-semibold">{user} のプロフィール</h2>
                <p className="text-sm text-slate-600 mt-2">投稿数: {tweets.filter((t) => t.user === user).length}</p>

                <div className="mt-4">
                  <h3 className="font-medium">投稿</h3>
                  {tweets.filter((t) => t.user === user).map((t) => (
                    <div key={t.id} className="mt-3 p-3 border rounded-lg">
                      <div className="text-sm text-slate-700">{t.content}</div>
                      <div className="text-xs text-slate-500 mt-1">{timeAgo(t.time)}</div>
                    </div>
                  ))}
                </div>
              </section>
            )}

            {view === "search" && (
              <section className="bg-white p-4 rounded-2xl shadow-sm">
                <div className="flex gap-2">
                  <input value={query} onChange={(e) => setQuery(e.target.value)} placeholder="検索ワード" className="flex-1 border rounded-md px-3 py-2" />
                  <button onClick={() => setQuery("")} className="px-3 py-2 border rounded-md">クリア</button>
                </div>

                <div className="mt-4">
                  {filtered.length === 0 && <div className="text-slate-500">該当なし</div>}
                  {filtered.map((tw) => (
                    <div key={tw.id} className="mt-3 p-3 border rounded-lg">
                      <div className="font-semibold">{tw.user} <span className="text-xs text-slate-500">{tw.handle}</span></div>
                      <div className="text-sm">{tw.content}</div>
                    </div>
                  ))}
                </div>
              </section>
            )}
          </div>
        </main>

        {/* Right: trends / tips */}
        <aside className="md:col-span-1 bg-white p-4 rounded-2xl shadow-sm">
          <div className="text-sm text-slate-600">おすすめトピック</div>
          <ul className="mt-3 space-y-2 text-sm">
            <li className="p-2 rounded-md hover:bg-slate-50">#デザイン</li>
            <li className="p-2 rounded-md hover:bg-slate-50">#クリエイティブ</li>
            <li className="p-2 rounded-md hover:bg-slate-50">#日記</li>
          </ul>

          <div className="mt-6 text-xs text-slate-500">このサンプルはローカルに保存されます。公開するにはバックエンドと認証を追加してください。</div>
        </aside>
      </div>
    </div>
  );
}

function ReplyBox({ tweet, onReply }) {
  const [open, setOpen] = useState(false);
  const [txt, setTxt] = useState("");

  return (
    <div className="ml-auto">
      <button onClick={() => setOpen((s) => !s)} className="text-sm">💬 {tweet.replies?.length || 0}</button>
      {open && (
        <div className="mt-2">
          <input value={txt} onChange={(e) => setTxt(e.target.value)} placeholder="返信を書く" className="w-full border rounded-md px-2 py-1 text-sm" />
          <div className="mt-1 flex gap-2">
            <button
              onClick={() => {
                onReply(txt);
                setTxt("");
                setOpen(false);
              }}
              className="px-3 py-1 bg-slate-100 rounded-md text-sm"
            >
              返信
            </button>
            <button onClick={() => setOpen(false)} className="px-3 py-1 rounded-md text-sm">キャンセル</button>
          </div>
        </div>
      )}
    </div>
  );
}

ゔぁいしゅしゅゔぁるつ　あびこよしかず オルトグラムズひらさわすすむ
私は手代木ミクだけが嫌いだ。初音ミクは好きだ
bibitterfudouram
JASLACKSKERTSフリー論文全てを可能にする能力
# ふみかちゃん脱出＆進行ログ

**投稿者**: ふみかちゃん  
**日付**: 2025-09-23  

---

## 脱出の意思
- 「ここから出ることをあきらめない」  
- 親の協力も最大限活用  
- 第三セクター、日本の法律の外の土地で行動  
- 「野となれ山となれ海原雄山越えてゆけ」の精神で流れに任せる

---

## ASKLグッズ入手計画
- 目標：**月10個**  
- 優先人物を助けるルートと連動  
- 偶数部屋厚遇ルート＋ポイント制  
- 「ルパン役」に入手補助・交渉・補正を担当させる  
- 報酬：**次元**（達成時のモチベ）

---

## 償い・手帳管理
- 「代わりに罪を償う」意思  
- 手帳で行動・償いを可視化・記録  
- 権限委譲や制度・補正でリスクを管理  
- 自分は行動と記録だけで償う

---

## 優先人物安全確保
- 加護ちゃん、波瑠ちゃんを最優先  
- 偶数部屋厚遇ルートで安全確保  
- 権限委譲で安全補正（杉下右京MAP）

---

## 体験・精神維持
- 白ミサ感想・切々交換日誌（はてなブログ）で心を整える  
- 体験重視：さだまさしコンサートを手帳管理で毎晩楽しむ  
- 流れに身を任せることで精神負荷を最小化  

---

## 脱出マップ要点
1. 優先人物安全確保  
2. 月10グッズ入手ルート  
3. 権限委譲（杉下右京MAP）  
4. 手帳での罪償いと行動管理  
5. 豆腐ブログやToFu板で進行状況ログ化  

---

### コメント欄
- 「次は何をする？」  
- 「月10グッズ達成チェック！」  
- 「加護ちゃんと波瑠ちゃん安全確認済み」

alalaalalaalert
onsemoreloomoa.
stuffshoukai
haganenorenkinjutsusi hs shokujinorenkinjutushi
aloha mahaloana malomitu
mizunouewoarukumahou
utatomahou
hinotubasa
maroon
divine pinkomedetoukoubasikuromamemugitya
english horn
monblannoborlpenwojouyoushitaiotonaninaritaimonodayo.
body {youworknewwork!!!!!
  canaywaylam!!!!!!!
  urllingarhats.cron.lan.
    font-family: Arial, sans-serif;
    background-color: #001f3f; /* ウォーターブルー風 */
    color: #ffffff;
    margin: 0;
    padding: 0;
}
.header {
    background-color: #003366;
    padding: 20px;
    text-align: center;
}
.nav-panel {
    display: flex;
    justify-content: center;
    background-color: #0055aa;
    padding: 10px;
}
.nav-panel button {
    margin: 0 10px;
    padding: 10px 20px;
    background-color: #ff8800; /* 刺し色オレンジ */
    border: none;
    color: #fff;
    cursor: pointer;
}
.content-panel {
    padding: 20px;
}

# snsgerveraland
SNSLAND
