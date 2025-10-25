# #10 정규식 테스터

> ⚠️ **개발 시작 전 필독!**
> 전역 개발 가이드: [`../_template/README.md`](../_template/README.md)
> Phase 1 버그, 체크리스트, 크로스 프로모션 도구 버튼 구현 확인 필수!

**URL:** regex.baal.co.kr

## 서비스 내용

정규식 실시간 테스트. 매치 하이라이트, 설명. 개발자용

## 기능 요구사항

- [ ] 정규식 입력 (플래그 지원)
- [ ] 테스트 문자열 입력 (여러 줄)
- [ ] 실시간 매칭 결과
- [ ] 매치된 부분 하이라이트
- [ ] 그룹 캡처 표시 ($1, $2 등)
- [ ] 플래그 옵션 (g, i, m, s, u, y)
- [ ] 매치 개수 표시
- [ ] 정규식 설명 생성 (자동)
- [ ] 자주 사용하는 패턴 예제
- [ ] 치환 기능 (Replace)
- [ ] 정규식 저장/불러오기

## 경쟁사 분석 (2025년 기준)

### 인기 사이트 TOP 5

1. **Regex101** - 가장 인기 있는 정규식 테스터
   - 강점: 실시간 매칭, 상세 설명, 디버거, 치트시트
   - 약점: 광고 있음, 복잡한 UI

2. **RegExr** - Adobe 제공
   - 강점: 시각적 표현, 예제 라이브러리, 커뮤니티
   - 약점: 로딩 느림

3. **Debuggex** - 시각화 특화
   - 강점: 정규식 플로우차트 시각화
   - 약점: 복잡한 정규식은 시각화 어려움

4. **RegexPal** - 심플한 UI
   - 강점: 빠르고 간단
   - 약점: 기능 제한적

5. **RegexTester** - 한국 서비스
   - 강점: 한글 지원
   - 약점: 기능 부족, 디자인 구식

### 우리의 차별화 전략

- ✅ **실시간 매칭** - 입력 즉시 결과 표시
- ✅ **그룹 캡처 시각화** - 캡처 그룹별 색상 구분
- ✅ **치환 기능** - Replace 미리보기
- ✅ **자주 쓰는 패턴** - 이메일, URL, 전화번호 등
- ✅ **다크모드** 지원
- ✅ **한/영 전환**
- ✅ **완전 무료** - 광고 없음

## 주요 라이브러리

### 옵션 1: 순수 JavaScript (추천!)

브라우저 내장 RegExp 사용

```javascript
// 기본 매칭
function testRegex(pattern, flags, text) {
  try {
    const regex = new RegExp(pattern, flags);
    const matches = [];

    if (flags.includes('g')) {
      // 전역 매칭 (모든 매치)
      let match;
      while ((match = regex.exec(text)) !== null) {
        matches.push({
          match: match[0],
          index: match.index,
          groups: match.slice(1), // 캡처 그룹
          input: match.input
        });
      }
    } else {
      // 단일 매칭
      const match = regex.exec(text);
      if (match) {
        matches.push({
          match: match[0],
          index: match.index,
          groups: match.slice(1),
          input: match.input
        });
      }
    }

    return {
      success: true,
      matches: matches,
      count: matches.length
    };
  } catch (error) {
    return {
      success: false,
      error: error.message
    };
  }
}

// 사용 예시
const result = testRegex(
  '(\\w+)@(\\w+)\\.com',  // 패턴
  'gi',                    // 플래그
  'test@example.com, hello@world.com'  // 텍스트
);

console.log(result);
// {
//   success: true,
//   matches: [
//     { match: 'test@example.com', index: 0, groups: ['test', 'example'] },
//     { match: 'hello@world.com', index: 19, groups: ['hello', 'world'] }
//   ],
//   count: 2
// }
```

### 하이라이트 표시

```javascript
function highlightMatches(text, matches) {
  let highlighted = '';
  let lastIndex = 0;

  // 인덱스 순으로 정렬
  matches.sort((a, b) => a.index - b.index);

  matches.forEach((match, i) => {
    // 매치 이전 텍스트
    highlighted += escapeHtml(text.substring(lastIndex, match.index));

    // 매치된 부분 (하이라이트)
    highlighted += `<mark class="match-${i % 5}">${escapeHtml(match.match)}</mark>`;

    lastIndex = match.index + match.match.length;
  });

  // 나머지 텍스트
  highlighted += escapeHtml(text.substring(lastIndex));

  return highlighted;
}

function escapeHtml(text) {
  return text
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#039;');
}
```

### 그룹 캡처 표시

```javascript
function displayCaptureGroups(matches) {
  const groups = [];

  matches.forEach((match, i) => {
    const groupData = {
      matchIndex: i,
      fullMatch: match.match,
      groups: []
    };

    match.groups.forEach((group, j) => {
      if (group !== undefined) {
        groupData.groups.push({
          index: j + 1,
          name: `$${j + 1}`,
          value: group
        });
      }
    });

    groups.push(groupData);
  });

  return groups;
}

// HTML 출력
function renderGroups(groups) {
  let html = '<div class="groups-list">';

  groups.forEach(g => {
    html += `<div class="match-group">`;
    html += `  <h4>Match ${g.matchIndex + 1}: ${g.fullMatch}</h4>`;
    html += `  <ul>`;

    g.groups.forEach(group => {
      html += `    <li><code>${group.name}</code>: ${group.value}</li>`;
    });

    html += `  </ul>`;
    html += `</div>`;
  });

  html += '</div>';
  return html;
}
```

### 치환 기능 (Replace)

```javascript
function replaceWithRegex(pattern, flags, text, replacement) {
  try {
    const regex = new RegExp(pattern, flags);
    const result = text.replace(regex, replacement);

    return {
      success: true,
      original: text,
      result: result,
      changed: text !== result
    };
  } catch (error) {
    return {
      success: false,
      error: error.message
    };
  }
}

// 사용 예시
const replaced = replaceWithRegex(
  '(\\w+)@(\\w+)\\.com',
  'gi',
  'Contact: test@example.com',
  '$1 [AT] $2 [DOT] com'  // 그룹 참조
);

console.log(replaced.result);
// "Contact: test [AT] example [DOT] com"
```

### 옵션 2: XRegExp (고급 기능)

Named capture groups 지원

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/xregexp/5.1.1/xregexp-all.min.js"></script>
```

```javascript
// Named groups
const regex = XRegExp('(?<user>\\w+)@(?<domain>\\w+)\\.com');
const match = XRegExp.exec('test@example.com', regex);

console.log(match.user);    // "test"
console.log(match.domain);  // "example"
```

## UI/UX 디자인 패턴

### 화면 구성

```
┌─────────────────────────────────────────────┐
│  정규식 테스터 (Regex Tester)                │
│  정규표현식을 실시간으로 테스트하세요          │
├─────────────────────────────────────────────┤
│  정규식 (Regular Expression):               │
│  ┌─────────────────────────────────────┐   │
│  │ (\w+)@(\w+)\.com                   │   │
│  └─────────────────────────────────────┘   │
│  플래그: ☑g ☑i ☐m ☐s ☐u ☐y               │
│                                             │
│  [자주 쓰는 패턴 ▼]                          │
│  • 이메일  • URL  • 전화번호  • 날짜         │
├─────────────────────────────────────────────┤
│  테스트 문자열:                              │
│  ┌─────────────────────────────────────┐   │
│  │ Contact us:                        │   │
│  │ john@example.com                   │   │
│  │ jane@company.com                   │   │
│  └─────────────────────────────────────┘   │
├─────────────────────────────────────────────┤
│  매칭 결과: 2개 매치됨 ✓                    │
│  ┌─────────────────────────────────────┐   │
│  │ Contact us:                        │   │
│  │ [john]@[example].com               │   │
│  │ [jane]@[company].com               │   │
│  └─────────────────────────────────────┘   │
│                                             │
│  캡처 그룹:                                 │
│  Match 1: john@example.com                 │
│    • $1: john                              │
│    • $2: example                           │
│  Match 2: jane@company.com                 │
│    • $1: jane                              │
│    • $2: company                           │
├─────────────────────────────────────────────┤
│  치환 (Replace):                            │
│  ┌─────────────────────────────────────┐   │
│  │ $1 [AT] $2 [DOT] com               │   │
│  └─────────────────────────────────────┘   │
│  결과:                                      │
│  ┌─────────────────────────────────────┐   │
│  │ Contact us:                        │   │
│  │ john [AT] example [DOT] com        │   │
│  │ jane [AT] company [DOT] com        │   │
│  └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

### 플래그 설명

| 플래그 | 이름 | 설명 |
|--------|------|------|
| g | global | 전역 매칭 (모든 매치 찾기) |
| i | ignore case | 대소문자 무시 |
| m | multiline | 여러 줄 모드 (^, $ 각 줄에 적용) |
| s | dotAll | . 이 줄바꿈 포함 |
| u | unicode | 유니코드 모드 |
| y | sticky | 마지막 인덱스부터 매칭 |

### 자주 쓰는 패턴 예제

```javascript
const commonPatterns = {
  email: {
    pattern: '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}',
    flags: 'gi',
    test: 'test@example.com, john.doe@company.co.kr',
    description: '이메일 주소'
  },
  url: {
    pattern: 'https?://[\\w.-]+\\.[a-zA-Z]{2,}(/[\\w.-]*)*',
    flags: 'gi',
    test: 'Visit https://example.com or http://test.co.kr',
    description: 'URL'
  },
  phone: {
    pattern: '0\\d{1,2}-\\d{3,4}-\\d{4}',
    flags: 'g',
    test: '010-1234-5678, 02-123-4567',
    description: '전화번호 (한국)'
  },
  date: {
    pattern: '\\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\\d|3[01])',
    flags: 'g',
    test: '2025-10-25, 2024-01-01',
    description: '날짜 (YYYY-MM-DD)'
  },
  ipv4: {
    pattern: '\\b(?:\\d{1,3}\\.){3}\\d{1,3}\\b',
    flags: 'g',
    test: '192.168.0.1, 127.0.0.1',
    description: 'IPv4 주소'
  },
  hexColor: {
    pattern: '#([0-9a-fA-F]{6}|[0-9a-fA-F]{3})',
    flags: 'gi',
    test: '#ff0000, #abc, #123456',
    description: 'HEX 색상 코드'
  }
};
```

## 난이도 & 예상 기간

- **난이도:** 쉬움
- **예상 기간:** 1일
- **실제 기간:** (작업 후 기록)

## 개발 일정

- [ ] 오전 1: UI 구성 (입력 필드, 플래그 체크박스)
- [ ] 오전 2: 정규식 매칭 로직, 실시간 업데이트
- [ ] 오후 1: 하이라이트 표시, 그룹 캡처
- [ ] 오후 2: 치환 기능, 자주 쓰는 패턴 예제
- [ ] 오후 3: 에러 처리, 최적화

## 트래픽 예상

⭐⭐ 중간 - 개발자 타겟 (필수 도구)

## SEO 키워드

- 정규식 테스트
- Regex Tester
- 정규표현식 테스터
- 정규식 연습
- Regular Expression
- 정규식 검증
- 정규식 예제

## 이슈 & 해결방안

### 실제 문제점 (경쟁사 분석 & 실무 이슈 기반)

1. **잘못된 정규식 입력 시 JavaScript 에러**
   - 원인: `new RegExp()` 생성자 예외 발생
   - 해결: try-catch로 에러 처리
   - 코드:
     ```javascript
     function safeRegex(pattern, flags) {
       try {
         return new RegExp(pattern, flags);
       } catch (error) {
         showError(`정규식 오류: ${error.message}`);
         return null;
       }
     }

     // 사용
     const regex = safeRegex(userPattern, userFlags);
     if (!regex) return; // 에러 시 중단
     ```

2. **전역 플래그(g) 사용 시 exec() 무한 루프**
   - 원인: `regex.lastIndex` 관리 실수
   - 해결: while 루프 안전장치
   - 코드:
     ```javascript
     function getAllMatches(regex, text) {
       const matches = [];
       let match;
       let iterations = 0;
       const MAX_ITERATIONS = 10000; // 안전장치

       while ((match = regex.exec(text)) !== null) {
         matches.push(match);

         // 무한 루프 방지
         if (++iterations > MAX_ITERATIONS) {
           showError('매칭이 너무 많습니다. 정규식을 확인하세요.');
           break;
         }

         // 빈 문자열 매칭 무한 루프 방지
         if (match.index === regex.lastIndex) {
           regex.lastIndex++;
         }
       }

       return matches;
     }
     ```

3. **대용량 텍스트에서 성능 저하**
   - 원인: 실시간 매칭으로 CPU 과부하
   - 해결: Debounce 적용 (300ms)
   - 코드:
     ```javascript
     let debounceTimer;

     function onInputChange() {
       clearTimeout(debounceTimer);

       debounceTimer = setTimeout(() => {
         performRegexTest();
       }, 300); // 300ms 후 실행
     }

     // 입력 이벤트
     patternInput.addEventListener('input', onInputChange);
     textInput.addEventListener('input', onInputChange);
     ```

4. **복잡한 정규식으로 인한 ReDoS (Regex Denial of Service)**
   - 원인: 백트래킹 과다 (catastrophic backtracking)
   - 해결: 타임아웃 설정
   - 코드:
     ```javascript
     function testRegexWithTimeout(pattern, flags, text, timeout = 1000) {
       return new Promise((resolve, reject) => {
         const worker = new Worker('regex-worker.js');

         const timer = setTimeout(() => {
           worker.terminate();
           reject(new Error('정규식 실행 시간 초과 (1초). 패턴을 단순화하세요.'));
         }, timeout);

         worker.onmessage = (e) => {
           clearTimeout(timer);
           worker.terminate();
           resolve(e.data);
         };

         worker.postMessage({ pattern, flags, text });
       });
     }
     ```

     **regex-worker.js:**
     ```javascript
     self.onmessage = (e) => {
       const { pattern, flags, text } = e.data;

       try {
         const regex = new RegExp(pattern, flags);
         const matches = [];
         let match;

         while ((match = regex.exec(text)) !== null) {
           matches.push({
             match: match[0],
             index: match.index,
             groups: match.slice(1)
           });
         }

         self.postMessage({ success: true, matches });
       } catch (error) {
         self.postMessage({ success: false, error: error.message });
       }
     };
     ```

5. **특수문자 이스케이프 미흡 (하이라이트 시 XSS 위험)**
   - 원인: HTML 특수문자 처리 안 함
   - 해결: escapeHtml 함수 사용
   - 코드:
     ```javascript
     function escapeHtml(text) {
       const div = document.createElement('div');
       div.textContent = text;
       return div.innerHTML;
     }

     // 또는 정규식으로
     function escapeHtml2(text) {
       return text
         .replace(/&/g, '&amp;')
         .replace(/</g, '&lt;')
         .replace(/>/g, '&gt;')
         .replace(/"/g, '&quot;')
         .replace(/'/g, '&#039;');
     }
     ```

6. **백슬래시 처리 문제 (\\d, \\w 등)**
   - 원인: JavaScript 문자열에서 백슬래시 이중 이스케이프
   - 해결: 사용자 입력은 그대로 사용
   - 코드:
     ```javascript
     // ❌ 잘못된 방법 (코드에서 직접 정의 시)
     const wrong = new RegExp("\\d+"); // 문자열 리터럴에서 \\d

     // ✅ 올바른 방법 (코드에서 직접 정의 시)
     const correct = new RegExp("\\\\d+"); // 이중 백슬래시
     // 또는
     const correct2 = /\d+/; // 정규식 리터럴 (백슬래시 한 번)

     // ✅ 사용자 입력은 그대로 사용
     const userPattern = patternInput.value; // "\d+" (사용자가 입력)
     const regex = new RegExp(userPattern); // 그대로 전달
     ```

7. **매칭 결과가 없을 때 사용자 혼란**
   - 원인: 빈 결과 표시 미흡
   - 해결: 명확한 메시지 표시
   - 코드:
     ```javascript
     function displayResults(matches) {
       if (matches.length === 0) {
         showMessage('매칭 결과 없음', 'warning');
         resultDiv.innerHTML = '<p class="no-match">매칭되는 텍스트가 없습니다.</p>';
       } else {
         showMessage(`${matches.length}개 매치됨`, 'success');
         resultDiv.innerHTML = highlightMatches(text, matches);
       }
     }
     ```

## 개발 로그

### 2025-10-25
- 프로젝트 폴더 생성
- **경쟁사 분석 완료:**
  - Regex101, RegExr, Debuggex, RegexPal, RegexTester 조사
  - Regex101이 가장 인기, 하지만 광고 있음
  - 차별화: 간단한 UI, 빠른 속도, 광고 없음
- **라이브러리 조사 완료:**
  - 순수 JavaScript RegExp (추천)
  - XRegExp (고급 기능 - named groups)
  - Best practices: try-catch, debounce, timeout
- **실제 이슈 파악:**
  - 잘못된 정규식 에러 처리
  - 무한 루프 방지
  - ReDoS 공격 방지 (타임아웃)
  - XSS 방지 (escapeHtml)
- **UI/UX 패턴:**
  - 3단 구성 (정규식 → 테스트 → 결과)
  - 실시간 매칭, 하이라이트
  - 그룹 캡처 표시, 치환 기능
  - 자주 쓰는 패턴 예제

## 참고 자료

- [RegExp - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp)
- [Regex Tutorial](https://regexone.com/)
- [Regex Cheat Sheet](https://www.debuggex.com/cheatsheet/regex/javascript)
- [ReDoS Attack](https://owasp.org/www-community/attacks/Regular_expression_Denial_of_Service_-_ReDoS)
- [XRegExp](http://xregexp.com/)
