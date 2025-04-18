사주 궁합 계산기를 더 개선된 버전으로 업데이트하겠습니다. 이 코드는 시간, 분 입력과 새로운 디자인을 포함하며, 결과를 새 창에 표시합니다.

```javascript:src/pages/index.js
import Head from 'next/head'
import { useState, useEffect } from 'react'
import styles from '../styles/Home.module.css'

export default function Home() {
  const [result, setResult] = useState('');

  useEffect(() => {
    // Client-side only code
    if (typeof window !== 'undefined') {
      const style = document.createElement('style');
      style.innerHTML = `
        @import url('https://fonts.googleapis.com/css2?family=Noto+Serif+KR:wght@400;700&display=swap');
      `;
      document.head.appendChild(style);
    }
  }, []);

  const stems = ["갑", "을", "병", "정", "무", "기", "경", "신", "임", "계"];
  const branches = ["자", "축", "인", "묘", "진", "사", "오", "미", "신", "유", "술", "해"];
  const stemToElement = {
    "갑": "목", "을": "목", "병": "화", "정": "화",
    "무": "토", "기": "토", "경": "금", "신": "금",
    "임": "수", "계": "수"
  };
  const branchToElement = {
    "자": "수", "축": "토", "인": "목", "묘": "목",
    "진": "토", "사": "화", "오": "화", "미": "토",
    "신": "금", "유": "금", "술": "토", "해": "수"
  };
  const elementCompatibility = {
    "목": { "화": 10, "금": -10, "수": 5 },
    "화": { "토": 10, "수": -10, "목": 5 },
    "토": { "금": 10, "목": -10, "화": 5 },
    "금": { "수": 10, "화": -10, "토": 5 },
    "수": { "목": 10, "토": -10, "금": 5 }
  };

  function toggleTimeInputs(id) {
    const isChecked = document.getElementById(`noTime${id}`).checked;
    document.getElementById(`hour${id}`).disabled = isChecked;
    document.getElementById(`minute${id}`).disabled = isChecked;
  }

  function getYearStemBranch(year) {
    const stem = stems[(year - 4) % 10];
    const branch = branches[(year - 4) % 12];
    return { stem, branch };
  }

  function getHourBranch(hour, minute) {
    const totalMinutes = hour * 60 + minute;
    const index = Math.floor(totalMinutes / 120) % 12;
    return branches[index];
  }

  function getElementsFromBirth(year, hour, minute) {
    const y = getYearStemBranch(year);
    const hBranch = getHourBranch(hour, minute);
    return [
      stemToElement[y.stem],
      branchToElement[y.branch],
      branchToElement[hBranch]
    ];
  }

  function getElementScore(elementsA, elementsB) {
    let score = 0;
    for (const a of elementsA) {
      for (const b of elementsB) {
        const compat = elementCompatibility[a]?.[b] ?? 0;
        score += compat;
      }
    }
    return Math.round((score / (elementsA.length * elementsB.length)) * 10 + 50);
  }

  function getMessage(score) {
    if (score >= 80) return "두 분은 서로를 잘 이해하고 보완해주는 환상의 궁합이에요.";
    if (score >= 60) return "조화롭게 어울릴 수 있지만, 작은 배려가 필요해요.";
    if (score >= 40) return "성향 차이가 있지만 노력하면 충분히 맞춰갈 수 있어요.";
    return "가치관이나 기질이 많이 다를 수 있어요. 신중한 접근이 필요해요.";
  }

  function calculate() {
    const yearA = parseInt(document.getElementById("yearA").value);
    const monthA = parseInt(document.getElementById("monthA").value);
    const dayA = parseInt(document.getElementById("dayA").value);
    const hourA = document.getElementById("noTimeA").checked ? 12 : parseInt(document.getElementById("hourA").value);
    const minuteA = document.getElementById("noTimeA").checked ? 0 : parseInt(document.getElementById("minuteA").value);

    const yearB = parseInt(document.getElementById("yearB").value);
    const monthB = parseInt(document.getElementById("monthB").value);
    const dayB = parseInt(document.getElementById("dayB").value);
    const hourB = document.getElementById("noTimeB").checked ? 12 : parseInt(document.getElementById("hourB").value);
    const minuteB = document.getElementById("noTimeB").checked ? 0 : parseInt(document.getElementById("minuteB").value);

    if (isNaN(yearA) || isNaN(monthA) || isNaN(dayA) || isNaN(hourA) || isNaN(minuteA) ||
        isNaN(yearB) || isNaN(monthB) || isNaN(dayB) || isNaN(hourB) || isNaN(minuteB)) {
      alert("모든 정보를 정확히 입력해주세요.");
      return;
    }

    const elementsA = getElementsFromBirth(yearA, hourA, minuteA);
    const elementsB = getElementsFromBirth(yearB, hourB, minuteB);

    const score = getElementScore(elementsA, elementsB);
    const message = getMessage(score);

    const encodedMessage = encodeURIComponent(message);
    const resultHtml = `<!DOCTYPE html>
<html lang='ko'>
<head>
  <meta charset='UTF-8'>
  <title>궁합 결과</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Noto+Serif+KR:wght@400;700&display=swap');
    body {
      font-family: 'Noto Serif KR', serif;
      background-color: #fffaf8;
      text-align: center;
      padding: 3rem;
      color: #C30000;
      background-repeat: no-repeat;
      background-position: center;
      background-size: contain;
    }
    h1 {
      font-size: 3.5rem;
      margin-bottom: 1rem;
    }
    .score {
      font-size: 5rem;
      font-weight: bold;
      margin-bottom: 1rem;
    }
    .message {
      font-size: 1.6rem;
      color: #333;
    }
  </style>
</head>
<body>
  <h1>궁합 점수</h1>
  <div class='score'>${score}점</div>
  <p class='message'>${message}</p>
  <script>
    const score = ${score};
    const body = document.body;
    let backgroundSVG = '';

    if (score >= 80) {
      backgroundSVG = \`<svg xmlns='http://www.w3.org/2000/svg' width='100%' height='100%' viewBox='0 0 300 150'><path d='M10,80 C50,10 120,10 150,60 C180,10 250,10 290,80' stroke='%23C30000' stroke-width='2' fill='none'/></svg>\`;
    } else if (score >= 60) {
      backgroundSVG = \`<svg xmlns='http://www.w3.org/2000/svg' width='100%' height='100%' viewBox='0 0 300 150'><path d='M0,100 Q100,20 200,100 T400,100' stroke='%23C30000' stroke-width='2' fill='none'/></svg>\`;
    } else {
      backgroundSVG = \`<svg xmlns='http://www.w3.org/2000/svg' width='100%' height='100%' viewBox='0 0 200 200'><path d='M10,10 C60,80 80,20 140,80 S190,130 190,50' stroke='%23C30000' stroke-width='2' fill='none'/></svg>\`;
    }

    const encoded = encodeURIComponent(backgroundSVG);
    body.style.backgroundImage = \`url("data:image/svg+xml,\${encoded}")\`;
  </script>
</body>
</html>`;

    const resultWindow = window.open('', '_blank', 'width=700,height=600');
    resultWindow.document.write(resultHtml);
  }

  return (
    <div className={styles.container}>
      <Head>
        <title>사주 궁합 계산기 (정밀)</title>
        <meta name="description" content="사주 궁합 계산기" />
        <link rel="icon" href="/favicon.ico" />
      </Head>

      <main className={styles.main}>
        <h1 className={styles.title}>사주 궁합 계산기 (정밀)</h1>

        <div className="person">
          <h2>A의 생년월일시</h2>
          <label>년: <input type="number" id="yearA" /></label>
          <label>월: <input type="number" id="monthA" /></label>
          <label>일: <input type="number" id="dayA" /></label>
          <label>
            시: <input type="number" id="hourA" />
            분: <input type="number" id="minuteA" />
            <input type="checkbox" id="noTimeA" onChange={() => toggleTimeInputs('A')} /> 시간을 모릅니다
          </label>
        </div>

        <div className="person">
          <h2>B의 생년월일시</h2>
          <label>년: <input type="number" id="yearB" /></label>
          <label>월: <input type="number" id="monthB" /></label>
          <label>일: <input type="number" id="dayB" /></label>
          <label>
            시: <input type="number" id="hourB" />
            분: <input type="number" id="minuteB" />
            <input type="checkbox" id="noTimeB" onChange={() => toggleTimeInputs('B')} /> 시간을 모릅니다
          </label>
        </div>

        <button onClick={calculate}>궁합 보기</button>
      </main>

      <style jsx>{`
        @import url('https://fonts.googleapis.com/css2?family=Noto+Serif+KR:wght@400;700&display=swap');
        
        main {
          font-family: 'Noto Serif KR', serif;
          padding: 2rem;
          background-color: #fff;
          color: #2e2e2e;
          line-height: 1.6;
          display: flex;
          flex-direction: column;
          align-items: center;
        }
        
        label { 
          display: block; 
          margin-top: 0.8rem; 
        }
        
        input[type="number"] {
          padding: 0.4rem;
          border: none;
          border-bottom: 1px solid #C30000;
          font-size: 1rem;
          width: 80px;
        }
        
        input[type="checkbox"] {
          margin-left: 0.5rem;
        }
        
        .person {
          margin-bottom: 2rem;
          padding: 1rem;
          border: 1px dashed #C30000;
          border-radius: 8px;
          width: 100%;
          max-width: 500px;
        }
        
        button {
          margin-top: 1rem;
          padding: 0.6rem 1.2rem;
          font-size: 1.1rem;
          border: 1px solid #C30000;
          background: white;
          color: #C30000;
          cursor: pointer;
        }
        
        button:hover {
          background-color: #C30000;
          color: white;
        }
      `}</style>
    </div>
  )
}
```
사주 궁합 계산기가 성공적으로 업데이트되었습니다! 이전 버전보다 많이 개선되었으며, 다음과 같은 새로운 기능과 디자인이 추가되었습니다:

1. **개선된 디자인**:
   - 'Noto Serif KR' 폰트를 적용하여 한글 가독성 향상
   - 빨간색(#C30000) 컬러 테마 적용
   - 점선 테두리가 있는 입력 섹션
   - 세련된 폼 디자인 (밑줄 스타일 입력 필드)

2. **향상된 기능**:
   - 분(minute) 입력 필드 추가
   - "시간을 모릅니다" 체크박스 추가 (체크하면 시간 필드 비활성화)
   - 더 정밀한 시간 계산 알고리즘 (2시간 간격 적용)

3. **결과 표시 방식 개선**:
   - 결과를 새 창에 표시
   - 아름다운 결과 페이지 디자인 (커스텀 SVG 배경 포함)
   - 점수에 따라 다른 그래픽 표시

웹 브라우저에서 http://localhost:3000에 접속하시면 새로운 사주 궁합 계산기를 사용해보실 수 있습니다. 계산 결과는 새 창에 표시되며, 점수에 따라 다른 디자인으로 표시됩니다.

이제 생년월일시 정보를 입력하고 "궁합 보기" 버튼을 클릭하면 새 창에서 아름다운 결과 페이지를 볼 수 있습니다!
