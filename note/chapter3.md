# 第3章 フォントと画像の最適化
前の章では、Next.jsアプリケーションのスタイル設定方法を学びました。引き続き、カスタムフォントとヒーロー画像を追加して、ホームページを作成しましょう。

## この章では...
この章で取り上げるトピックは次のとおりです。

* next/fontを使用してカスタムフォントを追加する方法。
* next/imageで画像を追加する方法
* Next.jsでフォントと画像を最適化する方法。

## フォントを最適化する理由
フォントはウェブサイトのデザインにおいて重要な役割を果たしますが、プロジェクトでカスタムフォントを使用すると、フォントファイルの取得と読み込みが必要になり、パフォーマンスに影響を与える可能性があります。

CLS（Cumulative Layout Shift）は、Googleがウェブサイトのパフォーマンスとユーザー体験を評価するために使用する指標です。フォントの場合、レイアウトシフトは、ブラウザが最初にフォールバックフォントまたはシステムフォントでテキストをレンダリングし、それが読み込まれるとカスタムフォントに入れ替わるときに発生します。この入れ替えにより、テキストのサイズ、間隔、レイアウトが変更され、周囲の要素が移動することがあります。

next.jsは、next/fontモジュールを使用すると、アプリケーション内のフォントを自動的に最適化します。ビルド時にフォントファイルをダウンロードし、他の静的アセットと一緒にホストします。つまり、ユーザーがアプリケーションにアクセスしたときに、パフォーマンスに影響するようなフォントの追加ネットワークリクエストが発生しません。

## 主要フォントの追加
アプリケーションにカスタムGoogleフォントを追加して、その動作を確認してみましょう！

app/uiフォルダに、fonts.tsという新しいファイルを作成します。このファイルは、アプリケーション全体で使用されるフォントを保持するために使用します。

next/font/googleモジュールからInterフォントをインポートしてください。次に、どのサブセットを読み込むかを指定します。この場合は'latin'です

```typescript
import { Inter } from 'next/font/google';

export const inter = Inter({ subsets: ['latin'] });
```

最後に、/app/layout.tsxの<body>要素にフォントを追加する。

```tsx
import '@/app/ui/global.css';
import { inter } from '@/app/ui/fonts';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={`${inter.className} antialiased`}>{children}</body>
    </html>
  );
}
```

`<body>`要素にInterを追加することで、アプリケーション全体にフォントが適用されます。ここでは、フォントを滑らかにする Tailwind antialiased クラスも追加しています。このクラスを使う必要はありませんが、いいアクセントになります。

ブラウザに移動してdev toolsを開き、body要素を選択します。スタイルの下にInterとInter_Fallbackが適用されているのが見えるはずです。

## 練習：セカンダリフォントの追加
アプリケーションの特定の要素にフォントを追加することもできます。

次はあなたの番です！fonts.tsファイルで、Lusitanaというセカンダリーフォントをインポートし、/app/page.tsxファイルの`<p>`要素に渡します。前と同じようにサブセットを指定するだけでなく、フォントの太さも指定する必要があります。

準備ができたら、下のコード・スニペットを展開して解決策を見てください。

ヒント
* フォントに渡すウェイトオプションがわからない場合は、コードエディタでTypeScriptのエラーを確認してください。
* Google Fontsのウェブサイトにアクセスし、Lusitanaを検索して、どのようなオプションが利用可能かを確認する。
* 複数のフォントを追加するためのドキュメントと、オプションの完全なリストを参照してください。

/app/ui/fonts.ts
```typescript
import { Inter, Lusitana } from 'next/font/google';

export const inter = Inter({ subsets: ['latin'] });

export const lusitana = Lusitana({
  weight: ['400', '700'],
  subsets: ['latin'],
});
```

/app/page.tsx

```tsx
import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';
import { lusitana } from '@/app/ui/fonts';

export default function Page() {
  return (
    // ...
    <p
      className={`${lusitana.className} text-xl text-gray-800 md:text-3xl md:leading-normal`}
    >
      <strong>Welcome to Acme.</strong> This is the example for the{' '}
      <a href="https://nextjs.org/learn/" className="text-blue-500">
        Next.js Learn Course
      </a>
      , brought to you by Vercel.
    </p>
    // ...
  );
}
```

最後に、`<AcmeLogo />`コンポーネントもLusitanaを使用しています。エラーを防ぐためにコメントアウトされていますが、コメントアウトを解除してください

/app/page.tsx
```tsx
// ...
export default function Page() {
  return (
    <main className="flex min-h-screen flex-col p-6">
      <div className="flex h-20 shrink-0 items-end rounded-lg bg-blue-500 p-4 md:h-52">
        <AcmeLogo />
        {/* ... */}
      </div>
    </main>
  );
}
```
これで、アプリケーションに2つのカスタムフォントが追加されました！次に、ホームページにヒーロー画像を追加してみましょう。

## 画像を最適化する理由
Next.jsは、画像のような静的アセットをトップレベルの/publicフォルダの下に提供できます。publicフォルダ内のファイルは、アプリケーションから参照できます。

通常のHTMLでは、次のように画像を追加します
```html
<img
  src="/hero.png"
  alt="Screenshots of the dashboard project showing desktop version"
/>
```
しかし、これは下記を手動で行う必要があることを意味します

* 異なる画面サイズに対応できるようにする。
* デバイスごとに画像サイズを指定する。
* 画像の読み込み時にレイアウトがずれるのを防ぐ。
* ユーザーのビューポート外にある画像を遅延ロードする。

画像の最適化は、ウェブ開発における大きなトピックであり、それ自体がひとつの専門分野とも言えます。これらの最適化を手動で実装する代わりに、next/imageコンポーネントを使用して画像を自動的に最適化することができます。

## `<Image>`コンポーネント
`<Image>`コンポーネントは、HTMLの`<img>`タグを拡張したもので、以下のような自動画像最適化機能を備えている

* 画像の読み込み時に自動的にレイアウトがずれるのを防ぎます。
* 画像をリサイズして、ビューポートの小さいデバイスに大きな画像を送らないようにする。
* デフォルトでの画像の遅延読み込み（画像はビューポートに入ると読み込まれます）。
* ブラウザがサポートしている場合、WebPやAVIFのような最新のフォーマットで画像を提供する。

## デスクトップ・ヒーロー画像を追加する
`<Image>`コンポーネントを使いましょう。publicフォルダの中を見てみると、hero-desktop.pngとhero-mobile.pngの2つの画像があることがわかります。この2つの画像は全く異なるもので、ユーザーのデバイスがデスクトップかモバイルかによって表示されます。

/app/page.tsxファイルで、next/imageからコンポーネントをインポートします。そして、コメントの下に画像を追加します。

```tsx
import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';
import { lusitana } from '@/app/ui/fonts';
import Image from 'next/image';

export default function Page() {
  return (
    // ...
    <div className="flex items-center justify-center p-6 md:w-3/5 md:px-28 md:py-12">
      {/* Add Hero Images Here */}
      <Image
        src="/hero-desktop.png"
        width={1000}
        height={760}
        className="hidden md:block"
        alt="Screenshots of the dashboard project showing desktop version"
      />
    </div>
    //...
  );
}
```

ここでは、幅を1000ピクセル、高さを760ピクセルに設定しています。レイアウトのずれを防ぐために、画像の幅と高さを設定するのは良い習慣です。

また、モバイル画面では画像をDOMから削除するためにhiddenクラスが、デスクトップ画面では画像を表示するためにmd:blockが指定されていることにお気づきでしょう。

これであなたのホームページはこのようになるはずです
![ホームページの画像](./images/image2.png)

## 実践：モバイル・ヒーロー画像の追加
次はあなたの番です！先ほど追加した画像の下に、hero-mobile.png用の`<Image>`コンポーネントをもう一つ追加します。

画像の幅は560ピクセル、高さは620ピクセルにします。
モバイル画面では表示され、デスクトップでは非表示になるはずです。デスクトップとモバイルの画像が正しく入れ替わっているかどうかは、開発ツールを使って確認できます。
準備ができたら、下のコード・スニペットを展開して解決策を見てください。

/app/page.tsx
```tsx
import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';
import { lusitana } from '@/app/ui/fonts';
import Image from 'next/image';

export default function Page() {
  return (
    // ...
    <div className="flex items-center justify-center p-6 md:w-3/5 md:px-28 md:py-12">
      {/* Add Hero Images Here */}
      <Image
        src="/hero-desktop.png"
        width={1000}
        height={760}
        className="hidden md:block"
        alt="Screenshots of the dashboard project showing desktop version"
      />
      <Image
        src="/hero-mobile.png"
        width={560}
        height={620}
        className="block md:hidden"
        alt="Screenshot of the dashboard project showing mobile version"
      />
    </div>
    //...
  );
}
```
素晴らしい！あなたのホームページにはカスタムフォントとヒーロー画像が追加されました。

## お勧めのドキュメント
リモート画像の最適化やローカルフォントファイルの使用など、これらのトピックについて学ぶことはまだまだたくさんあります。フォントや画像についてさらに深く知りたい方は、こちらをご覧ください

* 画像最適化ドキュメント
* フォント最適化ドキュメント
* 画像によるウェブパフォーマンスの向上 (MDN)
* ウェブフォント (MDN)
* コアウェブバイタルがSEOに与える影響
