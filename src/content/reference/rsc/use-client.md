---
title: "'use client'"
titleForTitleTag: "'use client' directive"
canary: true
---

<Canary>

`'use client'` hanya diperlukan jika Anda [menggunakan Komponen Server React](/learn/start-a-new-react-project#bleeding-edge-react-frameworks) atau sedang membangun library yang kompatibel dengan fitur tersebut.
</Canary>


<Intro>

`'use client'` memungkinkan Anda menandai kode yang dijalankan di sisi klien.

</Intro>

<InlineToc />

---

## Referensi {/*reference*/}

### `'use client'` {/*use-client*/}

Tambahkan `'use client'` di bagian atas sebuah *file* untuk menandai modul dan semua dependensi transitifnya sebagai kode klien.

```js {1}
'use client';

import { useState } from 'react';
import { formatDate } from './formatters';
import Button from './button';

export default function RichTextEditor({ timestamp, text }) {
  const date = formatDate(timestamp);
  // ...
  const editButton = <Button />;
  // ...
}
```

Ketika sebuah *file* ditandai dengan `'use client'` dan diimpor dari Komponen Server, [*bundler* yang kompatibel](/learn/start-a-new-react-project#bleeding-edge-react-frameworks)  akan memperlakukan impor modul tersebut sebagai batas antara kode yang dijalankan di server dan kode yang dijalankan di klien.

Sebagai dependensi dari *`RichTextEditor`*, *`formatDate`* dan *`Button`* juga akan dievaluasi di klien, terlepas dari apakah modulnya berisi direktif `'use client'` atau tidak. Perlu dicatat bahwa satu modul bisa dievaluasi di server jika diimpor dari kode server, dan di klien jika diimpor dari kode klien.

#### Peringatan {/*caveats*/}

* `'use client'` harus berada di bagian paling atas *file*, di atas semua pernyataan impor atau kode lainnya (komentar diperbolehkan). `'use client'` harus ditulis menggunakan tanda kutip tunggal ataupun ganda, tetapi tidak boleh menggunakan tanda petik terbalik.
* Ketika modul `'use client'` di impor dari modul lain yang juga dirender di sisi klien, direktif tersebut tidak memberikan efek apa pun.
* Ketika sebuah modul komponen memiliki direktif `'use client'`, setiap penggunaan komponen dari modul tersebut dijamin akan menjadi Komponen Klien. Namun, sebuah komponen tetap dapat dievaluasi di sisi klien meskipun tidak memiliki direktif `'use client'`.
  * Penggunaan komponen dianggap sebagai Komponen Klien jika komponen tersebut didefinisikan dalam modul yang memiliki direktif `'use client'`, atau merupakan dependensi transitif dari modul yang memiliki direktif `'use client'`. Selain itu, komponen dianggap sebagai Komponen Server.
* Kode yang dievaluasi di sisi klien tidak terbatas hanya pada komponen. Semua kode yang termasuk dalam sub-pohon modul Klien akan dikirim ke klien dan dijalankan di sisi klien.
* Ketika sebuah modul yang dievaluasi di sisi server mengimpor nilai dari modul `'use client'`, nilai-nilai tersebut harus berupa komponen React atau [nilai properti yang dapat diserialisasi](#passing-props-from-server-to-client-components) yang dapat diteruskan ke Komponen Klien. Penggunaan di luar itu akan menghasilkan pengecualian (*exception*).
  
### Cara `'use client'` menandai kode klien {/*how-use-client-marks-client-code*/}

Dalam aplikasi React, komponen biasanya dipisahkan ke dalam beberapa file, atau [modul](/learn/importing-and-exporting-components#exporting-and-importing-a-component).

Untuk aplikasi yang menggunakan Komponen Server React, tampilan aplikasi secara default dirender di sisi server. `'use client'` memperkenalkan batas antara server dan klien dalam [pohon dependensi modul](/learn/understanding-your-ui-as-a-tree#the-module-dependency-tree), dan secara efektif membentuk sub-pohon modul Klien.

Untuk memperjelas hal ini, perhatikan contoh aplikasi Komponen Server React berikut.

<Sandpack>

```js src/App.js
import FancyText from './FancyText';
import InspirationGenerator from './InspirationGenerator';
import Copyright from './Copyright';

export default function App() {
  return (
    <>
      <FancyText title text="Get Inspired App" />
      <InspirationGenerator>
        <Copyright year={2004} />
      </InspirationGenerator>
    </>
  );
}

```

```js src/FancyText.js
export default function FancyText({title, text}) {
  return title
    ? <h1 className='fancy title'>{text}</h1>
    : <h3 className='fancy cursive'>{text}</h3>
}
```

```js src/InspirationGenerator.js
'use client';

import { useState } from 'react';
import inspirations from './inspirations';
import FancyText from './FancyText';

export default function InspirationGenerator({children}) {
  const [index, setIndex] = useState(0);
  const quote = inspirations[index];
  const next = () => setIndex((index + 1) % inspirations.length);

  return (
    <>
      <p>Your inspirational quote is:</p>
      <FancyText text={quote} />
      <button onClick={next}>Inspire me again</button>
      {children}
    </>
  );
}
```

```js src/Copyright.js
export default function Copyright({year}) {
  return <p className='small'>©️ {year}</p>;
}
```

```js src/inspirations.js
export default [
  "Don’t let yesterday take up too much of today.” — Will Rogers",
  "Ambition is putting a ladder against the sky.",
  "A joy that's shared is a joy made double.",
];
```

```css
.fancy {
  font-family: 'Georgia';
}
.title {
  color: #007AA3;
  text-decoration: underline;
}
.cursive {
  font-style: italic;
}
.small {
  font-size: 10px;
}
```

</Sandpack>

Dalam pohon dependensi modul pada contoh aplikasi ini, direktif `'use client'` di `InspirationGenerator.js` menandai modul tersebut beserta semua dependensi transitifnya sebagai modul Klien. Subpohon yang dimulai dari `InspirationGenerator.js` sekarang ditandai sebagai modul Klien.

<Diagram name="use_client_module_dependency" height={250} width={545} alt="Graf pohon dengan simpul teratas mewakili modul 'App.js'. 'App.js' memiliki tiga anak: 'Copyright.js', 'FancyText.js', dan 'InspirationGenerator.js'. 'InspirationGenerator.js' memiliki dua anak: 'FancyText.js' dan 'inspirations.js'. Semua simpul di bawah (termasuk) 'InspirationGenerator.js' diberi latar kuning untuk menandakan bahwa sub‑graf ini di-render di sisi klien karena direktif 'use client' di 'InspirationGenerator.js'."> `'use client'` membagi pohon dependensi modul aplikasi Komponen Server React, menandai `InspirationGenerator.js` beserta seluruh dependensinya agar dirender di sisi klien. </Diagram>

Selama proses *render*, *framework* terlebih dahulu me-*render* komponen akar di sisi server dan menelusuri [pohon *render*](/learn/understanding-your-ui-as-a-tree#the-render-tree) sambil melewati evaluasi kode apa pun yang diimpor dari modul bertanda klien.

Bagian pohon *render* yang telah di-*render* di server kemudian dikirim ke klien. Setelah kode klien diunduh, sisi klien menyelesaikan pe-*render*-an sisa pohon.

<Diagram name="use_client_render_tree" height={250} width={500} alt="Graf pohon tempat setiap simpul mewakili sebuah komponen dan anak‑anaknya. Simpul teratas berlabel 'App' dan memiliki dua anak: 'InspirationGenerator' dan 'FancyText'. 'InspirationGenerator' memiliki dua anak: 'FancyText' dan 'Copyright'. Baik 'InspirationGenerator' maupun anaknya 'FancyText' ditandai untuk dirender di sisi klien."> Pohon render untuk aplikasi Komponen Server React. `InspirationGenerator` dan komponen anaknya `FancyText` diekspor dari kode bertanda klien dan dianggap sebagai Komponen Klien. </Diagram>

Kami memperkenalkan definisi berikut:

* **Komponen Klien** adalah komponen dalam pohon *render* yang di-*render* di sisi klien.
* **Komponen Server** adalah komponen dalam pohon *render* yang di-*render* di sisi server.

Berdasarkan contoh aplikasi, `App`, `FancyText`, dan `Copyright` di-*render* di sisi server dan dianggap sebagai Komponen Server. Karena `InspirationGenerator.js` dan semua dependensi transitifnya ditandai sebagai kode klien, maka komponen `InspirationGenerator` dan komponen anaknya `FancyText` adalah Komponen Klien.

<DeepDive>

#### Bagaimana mungkin `FancyText` merupakan Komponen Server sekaligus Komponen Klien? {/*how-is-fancytext-both-a-server-and-a-client-component*/}

Berdasarkan definisi di atas, komponen `FancyText` merupakan Komponen Server dan Komponen Klien. Bagaimana bisa?

Pertama, mari kita perjelas bahwa istilah “komponen” sebenarnya tidak terlalu spesifik. Berikut dua cara umum dalam memahami istilah “komponen”:

1. “Komponen” bisa merujuk pada **definisi komponen**. Dalam banyak kasus, ini bisa berarti fungsi.
```js
// This is a definition of a component
function MyComponent() {
  return <p>My Component</p>
}
```

2. “Komponen” juga dapat merujuk pada **penggunaan komponen** dari definisinya.
```js
import MyComponent from './MyComponent';

function App() {
  // This is a usage of a component
  return <MyComponent />;
}
```

Sering kali, ketidakpresisian ini tidak penting saat menjelaskan konsep. Namun, dalam kasus ini penting.

Saat kita membahas Komponen Server atau Komponen Klien, maksudnya adalah penggunaan komponen.

* Jika komponen didefinisikan dalam modul yang memiliki direktif `'use client'`, atau komponen tersebut diimpor dan dipanggil di dalam Komponen Klien, maka penggunaan komponen tersebut dianggap sebagai Komponen Klien.
* Jika tidak, penggunaan komponen tersebut dianggap sebagai Komponen Server.

<Diagram name="use_client_render_tree" height={150} width={450} alt="Graf pohon tempat setiap simpul mewakili sebuah komponen dan anak‑anaknya. Simpul teratas berlabel 'App' dan memiliki dua anak: 'InspirationGenerator' dan 'FancyText'. 'InspirationGenerator' memiliki dua anak: 'FancyText' dan 'Copyright'. Baik 'InspirationGenerator' maupun anaknya 'FancyText' ditandai untuk dirender di sisi klien.">Pohon render menggambarkan penggunaan komponen.</Diagram>

Kembali ke pertanyaan tentang `FancyText`, kita melihat bahwa definisi komponennya tidak memiliki direktif `'use client'` dan memiliki dua penggunaan.

Penggunaan `FancyText` sebagai anak dari App menandai penggunaan tersebut sebagai Komponen Server. Ketika `FancyText` diimpor dan dipanggil di dalam `InspirationGenerator`, 
penggunaan `FancyText` tersebut adalah Komponen Klien karena `InspirationGenerator` berisi direktif `'use client'`.

Ini berarti bahwa definisi komponen `FancyText` akan dievaluasi di sisi server dan juga diunduh oleh klien untuk merender penggunaannya sebagai Komponen Klien.

</DeepDive>

<DeepDive>

#### Kenapa `Copyright` merupakan Komponen Server? {/*why-is-copyright-a-server-component*/}
Karena `Copyright` di-*render* sebagai anak dari Komponen Klien `InspirationGenerator`, Anda mungkin terkejut mengetahui bahwa `Copyright` adalah sebuah Komponen Server.

Ingat bahwa `'use client'` menetapkan batas antara kode server dan kode klien berdasarkan pohon dependensi modul, bukan pohon *render*.

<Diagram name="use_client_module_dependency" height={200} width={500} alt="Graf pohon dengan simpul teratas mewakili modul 'App.js'. 'App.js' memiliki tiga anak: 'Copyright.js', 'FancyText.js', dan 'InspirationGenerator.js'. 'InspirationGenerator.js' memiliki dua anak: 'FancyText.js' dan 'inspirations.js'. Semua simpul di bawah (termasuk) 'InspirationGenerator.js' diberi latar kuning untuk menandakan bahwa sub‑graf ini di-render di sisi klien karena direktif 'use client' di 'InspirationGenerator.js'.">
`'use client'` menetapkan batas antara kode server dan kode klien pada pohon dependensi modul.
</Diagram>

In the module dependency tree, we see that `App.js` imports and calls `Copyright` from the `Copyright.js` module. As `Copyright.js` does not contain a `'use client'` directive, the component usage is rendered on the server. `App` is rendered on the server as it is the root component.

Client Components can render Server Components because you can pass JSX as props. In this case, `InspirationGenerator` receives `Copyright` as [children](/learn/passing-props-to-a-component#passing-jsx-as-children). However, the `InspirationGenerator` module never directly imports the `Copyright` module nor calls the component, all of that is done by `App`. In fact, the `Copyright` component is fully executed before `InspirationGenerator` starts rendering.

The takeaway is that a parent-child render relationship between components does not guarantee the same render environment.

Pada pohon dependensi modul, kita melihat bahwa `App.js` mengimpor dan memanggil `Copyright` dari modul `Copyright.js`. Karena `Copyright.js` tidak memiliki direktif `'use client'`, maka penggunaan komponen tersebut di-*render* di sisi server. `App` juga di-*render* di server karena merupakan komponen akar.

Komponen Klien dapat merender Komponen Server karena Anda dapat meneruskan JSX sebagai *props*. Dalam kasus ini, `InspirationGenerator` menerima `Copyright` sebagai [anak](/learn/passing-props-to-a-component#passing-jsx-as-children). Namun, modul `InspirationGenerator` tidak pernah secara langsung mengimpor atau memanggil modul `Copyright`, semua itu dilakukan oleh `App`. Faktanya, komponen `Copyright` sudah dieksekusi sepenuhnya sebelum `InspirationGenerator` mulai di-*render*.

Kesimpulannya, hubungan induk-anak dalam pohon *render* tidak menjamin bahwa komponen-komponen tersebut berada di lingkungan *render* yang sama.

</DeepDive>

### Kapan saat yang tepat menggunakan `'use client'` {/*when-to-use-use-client*/}

Dengan `'use client'`, Anda dapat menentukan kapan sebuah komponen menjadi Komponen Klien. Karena secara bawaan komponen adalah Komponen Server, berikut adalah ringkasan singkat mengenai keuntungan dan keterbatasan Komponen Server untuk membantu Anda menentukan kapan sebuah komponen perlu ditandai agar dijalankan di sisi klien.

Untuk penyederhanaan, kami menggunakan istilah Komponen Server, namun prinsip yang sama juga berlaku untuk seluruh kode dalam aplikasi Anda yang dijalankan di server.

#### Keuntungan Komponen Server {/*advantages*/}
* Komponen Server dapat mengurangi jumlah kode yang dikirim dan dijalankan oleh klien. Hanya modul Klien yang akan dibundel dan dievaluasi oleh klien.
* Komponen Server mendapat manfaat dari eksekusi di sisi server. Mereka dapat mengakses *filesystem* lokal dan berpotensi memiliki latensi rendah dalam pengambilan data dan permintaan jaringan.

#### Keterbatasan Komponen Server {/*limitations*/}
* Komponen Server tidak dapat menangani interaksi karena *event handler* harus didaftarkan dan dipicu oleh klien.
  * Misalnya, *event handler* seperti `onClick` hanya dapat didefinisikan di Komponen Klien.
* Komponen Server tidak dapat menggunakan sebagian besar Hook.
  * Ketika Komponen Server di-*render*, hasilnya adalah daftar komponen untuk di-*render* oleh klien. Komponen Server tidak tersimpan di memori setelah di-*render* dan tidak dapat memiliki *state*-nya sendiri.

### Tipe data yang dapat diserialisasi dan dikembalikan oleh Komponen Server {/*serializable-types*/}

Seperti pada aplikasi React lainnya, komponen induk meneruskan data ke komponen anak. Karena komponen-komponen ini dirender di lingkungan yang berbeda, meneruskan data dari Komponen Server ke Komponen Klien memerlukan perhatian khusus.

Nilai *prop* yang dikirim dari Komponen Server ke Komponen Klien harus dapat diserialisasi.

*Prop* yang dapat diserialisasi meliputi:

* Primitif
	* [*string*](https://developer.mozilla.org/en-US/docs/Glossary/String)
	* [*number*](https://developer.mozilla.org/en-US/docs/Glossary/Number)
	* [*bigint*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt)
	* [*boolean*](https://developer.mozilla.org/en-US/docs/Glossary/Boolean)
	* [*undefined*](https://developer.mozilla.org/en-US/docs/Glossary/Undefined)
	* [*null*](https://developer.mozilla.org/en-US/docs/Glossary/Null)
	* [Simbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol), hanya simbol yang didaftarkan dalam Registri Simbol Global melalui [`Symbol.for`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/for)
* *Iterable* yang berisi nilai yang dapat diserialkan
	* [*String*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)
	* [Senarai](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
	* [*Map*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
	* [*Set*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)
	* [*TypedArray*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) dan [*ArrayBuffer*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)
* [*Date*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)
* [Objek](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object) biasa: objek yang dibuat dengan [*object initializers*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer), dengan properti yang dapat diserialisasi
* Fungsi yang merupakan [Aksi Server](/reference/rsc/use-server)
* Elemen Komponen Klien atau Komponen Server (JSX)
* [*Promises*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

Berikut adalah tipe data yang tidak didukung:

* [Fungsi](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function) yang tidak diekspor dari modul yang ditandai sebagai modul klien atau tidak ditandai dengan [`'use server'`](/reference/rsc/use-server)
* [Kelas](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Classes_in_JavaScript)
* Objek yang merupakan *instance* dari kelas apa pun (selain bawaan seperti yang telah disebutkan) atau objek dengan [*null prototype*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object#null-prototype_objects)
* Simbol yang tidak didaftarkan secara global, misalnya `Symbol('my new symbol')`

## Penggunaan {/*usage*/}

### Building with interactivity and state {/*building-with-interactivity-and-state*/}

<Sandpack>

```js src/App.js
'use client';

import { useState } from 'react';

export default function Counter({initialValue = 0}) {
  const [countValue, setCountValue] = useState(initialValue);
  const increment = () => setCountValue(countValue + 1);
  const decrement = () => setCountValue(countValue - 1);
  return (
    <>
      <h2>Count Value: {countValue}</h2>
      <button onClick={increment}>+1</button>
      <button onClick={decrement}>-1</button>
    </>
  );
}
```

</Sandpack>

As `Counter` requires both the `useState` Hook and event handlers to increment or decrement the value, this component must be a Client Component and will require a `'use client'` directive at the top.

`Counter` membutukan hook `useState` dan *event handler* untuk menambah dan mengurangi nilai, component `Counter` harus Komponen Klien dan wajib menggunakan direktif `use client` di atas *file*

In contrast, a component that renders UI without interaction will not need to be a Client Component.

Komponen yang me-*render* UI tanpa interaksi tidak perlu menjadi Komponen Klien

```js
import { readFile } from 'node:fs/promises';
import Counter from './Counter';

export default async function CounterContainer() {
  const initialValue = await readFile('/path/to/counter_value');
  return <Counter initialValue={initialValue} />
}
```

For example, `Counter`'s parent component, `CounterContainer`, does not require `'use client'` as it is not interactive and does not use state. In addition, `CounterContainer` must be a Server Component as it reads from the local file system on the server, which is possible only in a Server Component.

There are also components that don't use any server or client-only features and can be agnostic to where they render. In our earlier example, `FancyText` is one such component.

```js
export default function FancyText({title, text}) {
  return title
    ? <h1 className='fancy title'>{text}</h1>
    : <h3 className='fancy cursive'>{text}</h3>
}
```

In this case, we don't add the `'use client'` directive, resulting in `FancyText`'s _output_ (rather than its source code) to be sent to the browser when referenced from a Server Component. As demonstrated in the earlier Inspirations app example, `FancyText` is used as both a Server or Client Component, depending on where it is imported and used.

But if `FancyText`'s HTML output was large relative to its source code (including dependencies), it might be more efficient to force it to always be a Client Component. Components that return a long SVG path string are one case where it may be more efficient to force a component to be a Client Component.

### Using client APIs {/*using-client-apis*/}

Your React app may use client-specific APIs, such as the browser's APIs for web storage, audio and video manipulation, and device hardware, among [others](https://developer.mozilla.org/en-US/docs/Web/API).

In this example, the component uses [DOM APIs](https://developer.mozilla.org/en-US/docs/Glossary/DOM) to manipulate a [`canvas`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/canvas) element. Since those APIs are only available in the browser, it must be marked as a Client Component.

```js
'use client';

import {useRef, useEffect} from 'react';

export default function Circle() {
  const ref = useRef(null);
  useLayoutEffect(() => {
    const canvas = ref.current;
    const context = canvas.getContext('2d');
    context.reset();
    context.beginPath();
    context.arc(100, 75, 50, 0, 2 * Math.PI);
    context.stroke();
  });
  return <canvas ref={ref} />;
}
```

### Using third-party libraries {/*using-third-party-libraries*/}

Often in a React app, you'll leverage third-party libraries to handle common UI patterns or logic.

These libraries may rely on component Hooks or client APIs. Third-party components that use any of the following React APIs must run on the client:
* [createContext](/reference/react/createContext)
* [`react`](/reference/react/hooks) and [`react-dom`](/reference/react-dom/hooks) Hooks, excluding [`use`](/reference/react/use) and [`useId`](/reference/react/useId)
* [forwardRef](/reference/react/forwardRef)
* [memo](/reference/react/memo)
* [startTransition](/reference/react/startTransition)
* If they use client APIs, ex. DOM insertion or native platform views

If these libraries have been updated to be compatible with React Server Components, then they will already include `'use client'` markers of their own, allowing you to use them directly from your Server Components. If a library hasn't been updated, or if a component needs props like event handlers that can only be specified on the client, you may need to add your own Client Component file in between the third-party Client Component and your Server Component where you'd like to use it.

[TODO]: <> (Troubleshooting - need use-cases)
