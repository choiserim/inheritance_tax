<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>법정 상속분 계산기</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100..900&family=Noto+Sans+KR:wght@100..900&display=swap');
        body {
            font-family: 'Noto Sans KR', 'Inter', sans-serif;
            background-color: #f7f7f7;
        }
        .step-card {
            background-color: #ffffff;
            border-radius: 0.75rem;
            padding: 1.5rem;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
            margin-bottom: 1.5rem;
        }
        .result-box {
            background-color: #ecfdf5; /* Tailwind green-50 */
            border: 2px solid #059669; /* Tailwind green-600 */
            border-radius: 0.75rem;
            padding: 1.5rem;
            margin-top: 2rem;
        }
        .input-group label {
            font-weight: 500;
        }
        .tooltip-container {
            position: relative;
            display: inline-block;
        }
        .tooltip {
            visibility: hidden;
            background-color: #333;
            color: #fff;
            text-align: center;
            border-radius: 6px;
            padding: 5px 10px;
            position: absolute;
            z-index: 10;
            bottom: 125%;
            left: 50%;
            margin-left: -150px;
            opacity: 0;
            transition: opacity 0.3s;
            width: 300px;
            font-size: 0.8rem;
        }
        .tooltip-container:hover .tooltip {
            visibility: visible;
            opacity: 1;
        }
    </style>
</head>
<body class="p-4 sm:p-8">

    <div class="max-w-4xl mx-auto">
        <header class="text-center mb-8">
            <h1 class="text-3xl sm:text-4xl font-extrabold text-blue-700">법정 상속분 계산기</h1>
            <p class="text-gray-600 mt-2">대한민국 민법에 따른 법정 상속 비율 및 금액을 계산합니다.</p>
        </header>

        <!-- 경고 메시지 -->
        <div class="bg-red-100 border-l-4 border-red-500 text-red-700 p-4 mb-6 rounded-lg" role="alert">
            <p class="font-bold">⚠️ 법적 고지</p>
            <p class="text-sm">이 계산기는 단순 참고용이며, 유류분, 기여분, 특별수익 등의 복잡한 법적 상황을 반영하지 않습니다. 법적 효력이 없으므로 정확한 상속을 위해서는 반드시 법률 전문가의 상담을 받으셔야 합니다.</p>
        </div>

        <!-- 1단계: 상속 재산 입력 -->
        <section class="step-card">
            <h2 class="text-2xl font-bold text-gray-800 mb-4 flex items-center">
                <span class="bg-blue-500 text-white rounded-full w-8 h-8 flex items-center justify-center mr-3 text-sm">1</span>
                상속 재산 입력 (순재산액)
            </h2>
            <div class="input-group">
                <label for="inheritanceAmount" class="block text-gray-700 mb-2">총 상속액 (원)</label>
                <input type="text" id="inheritanceAmount" placeholder="100,000,000"
                       class="w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 text-lg"
                       oninput="formatNumber(this)" value="500,000,000">
                <p class="text-sm text-gray-500 mt-1">부채를 제외한 순수한 재산액을 입력해 주세요. (예: 500,000,000)</p>
            </div>
        </section>

        <!-- 2단계: 상속인 선택 -->
        <section class="step-card">
            <h2 class="text-2xl font-bold text-gray-800 mb-4 flex items-center">
                <span class="bg-blue-500 text-white rounded-full w-8 h-8 flex items-center justify-center mr-3 text-sm">2</span>
                피상속인 (망자)의 상속인 구성 선택
            </h2>

            <div class="space-y-4">
                <div class="flex items-center space-x-6">
                    <label class="block text-gray-700 w-24">배우자</label>
                    <div class="flex items-center space-x-4">
                        <input type="radio" id="spouseExists" name="spouse" value="1" checked onchange="updateChildrenStatus()">
                        <label for="spouseExists">생존</label>
                        <input type="radio" id="spouseNone" name="spouse" value="0" onchange="updateChildrenStatus()">
                        <label for="spouseNone">없음</label>
                    </div>
                </div>

                <div class="flex items-center space-x-6">
                    <label for="childrenCount" class="block text-gray-700 w-24">자녀 수</label>
                    <input type="number" id="childrenCount" min="0" value="2"
                           class="w-24 p-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500">
                </div>

                <div id="parent-section" class="flex items-center space-x-6">
                    <label class="block text-gray-700 w-24">부모 (직계존속)</label>
                    <div class="flex items-center space-x-4">
                        <input type="radio" id="parentExists" name="parent" value="1" disabled>
                        <label for="parentExists">생존</label>
                        <input type="radio" id="parentNone" name="parent" value="0" checked disabled>
                        <label for="parentNone">없음</label>
                        <div class="tooltip-container">
                            <span class="text-blue-500 cursor-pointer text-sm">
                                [?]
                                <div class="tooltip">
                                    민법상 자녀가 1명이라도 있으면 부모는 상속인이 될 수 없습니다. 자녀가 모두 없는 경우에만 부모를 선택하세요.
                                </div>
                            </span>
                        </div>
                    </div>
                </div>

                <div id="sibling-section" class="flex items-center space-x-6">
                    <label for="siblingCount" class="block text-gray-700 w-24">형제자매 수</label>
                    <input type="number" id="siblingCount" min="0" value="0" disabled
                           class="w-24 p-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500">
                    <div class="tooltip-container">
                        <span class="text-blue-500 cursor-pointer text-sm">
                            [?]
                            <div class="tooltip">
                                형제자매는 자녀, 부모, 배우자가 모두 없는 경우에만 상속인이 될 수 있습니다.
                            </div>
                        </span>
                    </div>
                </div>
            </div>
        </section>

        <!-- 3단계: 계산 버튼 -->
        <div class="text-center mb-8">
            <button onclick="calculateInheritance()"
                    class="w-full sm:w-auto px-10 py-4 bg-green-600 text-white text-xl font-bold rounded-xl hover:bg-green-700 transition duration-300 shadow-lg transform hover:scale-105">
                상속분 계산하기
            </button>
        </div>

        <!-- 결과 표시 영역 -->
        <section id="result-area" class="result-box hidden">
            <h2 class="text-2xl font-bold text-green-700 mb-4 border-b border-green-300 pb-2">계산 결과</h2>

            <div id="summary" class="mb-6">
                <p class="text-lg font-semibold text-gray-800">총 상속 재산: <span id="totalAmount" class="text-blue-600">0원</span></p>
                <p class="text-lg font-semibold text-gray-800">총 법정 상속인 수 (배우자 가산 비율 적용): <span id="totalUnits" class="text-blue-600">0명분</span></p>
            </div>

            <h3 class="text-xl font-semibold text-gray-800 mb-3">상속인별 법정 상속분</h3>
            <div class="overflow-x-auto">
                <table class="min-w-full bg-white rounded-lg shadow-md">
                    <thead>
                        <tr class="bg-green-100 text-green-800">
                            <th class="py-3 px-4 text-left">구분</th>
                            <th class="py-3 px-4 text-left">법정 비율</th>
                            <th class="py-3 px-4 text-left">법정 상속분 (원)</th>
                        </tr>
                    </thead>
                    <tbody id="resultTableBody">
                        <!-- 결과가 여기에 삽입됩니다. -->
                    </tbody>
                </table>
            </div>
        </section>

    </div>

    <script>
        // 숫자 콤마 포맷 함수
        function formatNumber(input) {
            let value = input.value.replace(/,/g, '');
            if (isNaN(value)) {
                input.value = input.value.replace(/[^\d,]/g, '');
                return;
            }
            input.value = new Intl.NumberFormat('ko-KR').format(value);
        }

        // 입력 값 정리 함수
        function getNumberValue(id) {
            const input = document.getElementById(id).value;
            return parseInt(input.replace(/,/g, '')) || 0;
        }

        // 상속인 선택에 따른 부모/형제자매 선택 활성화/비활성화
        function updateChildrenStatus() {
            const childrenCount = parseInt(document.getElementById('childrenCount').value);
            const childrenExists = childrenCount > 0;
            const spouseExists = document.getElementById('spouseExists').checked;

            const parentExistsRadio = document.getElementById('parentExists');
            const parentNoneRadio = document.getElementById('parentNone');
            const siblingCountInput = document.getElementById('siblingCount');

            // 1. 부모/형제자매 입력 활성화/비활성화 로직 (민법 상속 순위)
            // 1순위: 직계비속(자녀) 및 배우자
            // 2순위: 직계존속(부모) 및 배우자 (1순위가 없을 때)
            // 3순위: 형제자매 (1, 2순위가 없을 때)

            if (childrenExists || spouseExists) {
                // 1순위 상속인이 있으면 2순위, 3순위 비활성화
                parentExistsRadio.disabled = true;
                parentNoneRadio.disabled = true;
                parentNoneRadio.checked = true; // 강제 '없음' 선택

                siblingCountInput.disabled = true;
                siblingCountInput.value = 0;
            } else {
                // 1순위 상속인 (자녀, 배우자)이 모두 없을 때
                // 2순위 (부모) 활성화
                parentExistsRadio.disabled = false;
                parentNoneRadio.disabled = false;

                if (parentExistsRadio.checked) {
                    // 부모가 있으면 3순위 (형제자매) 비활성화
                    siblingCountInput.disabled = true;
                    siblingCountInput.value = 0;
                } else {
                    // 부모가 없으면 3순위 (형제자매) 활성화
                    siblingCountInput.disabled = false;
                }
            }
        }

        // 계산 실행 함수
        function calculateInheritance() {
            const totalInheritance = getNumberValue('inheritanceAmount');
            const childrenCount = parseInt(document.getElementById('childrenCount').value) || 0;
            const spouseExists = document.getElementById('spouseExists').checked;
            const parentExists = !document.getElementById('parentExists').disabled && document.getElementById('parentExists').checked;
            const siblingCount = !document.getElementById('siblingCount').disabled ? parseInt(document.getElementById('siblingCount').value) || 0 : 0;

            let units = []; // 상속인별 상속분 단위 (1, 1.5 등)
            let resultTableBody = '';
            let totalUnits = 0;
            let primaryHeirFound = false;

            // 상속 순위 결정 및 단위 계산
            // 1순위: 직계비속 (자녀)
            if (childrenCount > 0) {
                for (let i = 0; i < childrenCount; i++) {
                    units.push({ type: '자녀', unit: 1 });
                }
                primaryHeirFound = true;
            }

            // 1순위와 동순위: 배우자 (직계비속이 있을 때)
            if (spouseExists && (childrenCount > 0 || parentExists)) {
                units.push({ type: '배우자', unit: 1.5 });
                primaryHeirFound = true;
            }

            // 2순위: 직계존속 (부모) (1순위가 없을 때)
            if (!primaryHeirFound && parentExists) {
                units.push({ type: '부모', unit: 1 }); // 부모는 1인당 1단위로 계산
                // 부모가 상속인이면 (배우자 포함) 3순위는 자동 제외
                primaryHeirFound = true;
            }

            // 3순위: 형제자매 (1, 2순위가 모두 없을 때)
            if (!primaryHeirFound && siblingCount > 0) {
                for (let i = 0; i < siblingCount; i++) {
                    units.push({ type: '형제자매', unit: 1 });
                }
                primaryHeirFound = true;
            }
            
            // 4순위: 4촌 이내 방계혈족 (이 계산기는 3순위까지만 처리)
            if (!primaryHeirFound && units.length === 0) {
                 // 최종적으로 아무도 상속인이 아니면 국가 귀속 등을 안내 (여기서는 0으로 처리)
            }


            // 총 상속분 단위 합산
            if (units.length > 0) {
                totalUnits = units.reduce((sum, heir) => sum + heir.unit, 0);
            } else {
                totalUnits = 0;
                resultTableBody = `<tr><td colspan="3" class="py-3 px-4 text-center text-red-500">법정 상속인이 없습니다.</td></tr>`;
            }

            // 개별 상속분 계산 및 테이블 생성
            const processedHeirs = {};
            units.forEach(heir => {
                const fraction = heir.unit / totalUnits;
                const amount = totalInheritance * fraction;

                if (processedHeirs[heir.type]) {
                    processedHeirs[heir.type].count++;
                    processedHeirs[heir.type].amount += amount;
                    processedHeirs[heir.type].unit += heir.unit;
                } else {
                    processedHeirs[heir.type] = {
                        count: 1,
                        amount: amount,
                        unit: heir.unit
                    };
                }
            });

            // 테이블 행 생성
            if (totalUnits > 0) {
                // 배우자 항목 분리
                const spouseData = processedHeirs['배우자'];
                if (spouseData) {
                    resultTableBody += `
                        <tr class="hover:bg-green-50">
                            <td class="py-3 px-4">배우자</td>
                            <td class="py-3 px-4">
                                ${spouseData.unit.toFixed(1)}/${totalUnits.toFixed(1)}
                            </td>
                            <td class="py-3 px-4 text-base font-semibold text-gray-800">
                                ${new Intl.NumberFormat('ko-KR').format(Math.round(spouseData.amount))}원
                            </td>
                        </tr>
                    `;
                    delete processedHeirs['배우자'];
                }
                
                // 자녀/부모/형제자매 항목 분리
                for (const type in processedHeirs) {
                    const data = processedHeirs[type];
                    // 개별 상속인 1인당 상속분 계산
                    const unitPerPerson = data.unit / data.count; 
                    const amountPerPerson = data.amount / data.count;

                    // 대표 상속인 (1인) 행
                    resultTableBody += `
                        <tr class="hover:bg-green-50">
                            <td class="py-3 px-4">${type} (1인당)</td>
                            <td class="py-3 px-4">
                                ${unitPerPerson.toFixed(1)}/${totalUnits.toFixed(1)}
                            </td>
                            <td class="py-3 px-4 text-base font-semibold text-gray-800">
                                ${new Intl.NumberFormat('ko-KR').format(Math.round(amountPerPerson))}원
                            </td>
                        </tr>
                    `;

                    // 총 상속분 행 (자녀가 2명 이상일 때만 표시)
                    if(data.count > 1) {
                         resultTableBody += `
                            <tr class="bg-gray-50 text-gray-600">
                                <td class="py-2 px-4 italic">${type} (총 ${data.count}명)</td>
                                <td class="py-2 px-4 italic">총 ${data.unit.toFixed(1)}/${totalUnits.toFixed(1)}</td>
                                <td class="py-2 px-4 italic text-sm">
                                    ${new Intl.NumberFormat('ko-KR').format(Math.round(data.amount))}원 (총액)
                                </td>
                            </tr>
                        `;
                    }
                }
            }


            // 최종 결과 출력
            document.getElementById('totalAmount').textContent = new Intl.NumberFormat('ko-KR').format(totalInheritance) + '원';
            document.getElementById('totalUnits').textContent = totalUnits.toFixed(1) + '명분';
            document.getElementById('resultTableBody').innerHTML = resultTableBody;
            document.getElementById('result-area').classList.remove('hidden');
        }

        // 초기 로드 시 한 번 실행
        window.onload = function() {
            updateChildrenStatus();
        }
        document.getElementById('childrenCount').addEventListener('input', updateChildrenStatus);
        document.getElementById('parentExists').addEventListener('change', updateChildrenStatus);
        document.getElementById('parentNone').addEventListener('change', updateChildrenStatus);

    </script>
</body>
</html>
