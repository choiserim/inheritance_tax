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
        /* 기존 select-custom 스타일 제거 (더 이상 사용하지 않음) */
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
                <!-- 배우자 (1순위/2순위 동순위) -->
                <div class="flex items-center space-x-6">
                    <label class="block text-gray-700 w-24">배우자</label>
                    <div class="flex items-center space-x-4">
                        <input type="radio" id="spouseExists" name="spouse" value="1" checked onchange="updateHeirInputs()">
                        <label for="spouseExists">생존</label>
                        <input type="radio" id="spouseNone" name="spouse" value="0" onchange="updateHeirInputs()">
                        <label for="spouseNone">없음</label>
                    </div>
                </div>

                <!-- 자녀 수 (1순위) - value="0"으로 수정됨 -->
                <div class="flex items-center space-x-6">
                    <label for="childrenCount" class="block text-gray-700 w-24">직계비속 (자녀 수)</label>
                    <input type="number" id="childrenCount" min="0" value="0"
                           class="w-24 p-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
                           oninput="updateHeirInputs()">
                </div>
                
                <!-- 직계존속 (부모) - 2순위 -->
                <div class="flex items-start space-x-6">
                    <label class="block text-gray-700 w-24 pt-2">직계존속 (부모)</label>
                    <div class="flex flex-col space-y-2" id="parent-control-group">
                        <!-- 1. 생존 여부 토글 -->
                        <div class="flex items-center space-x-4">
                            <!-- 이 라디오 버튼들은 항상 활성화 상태를 유지합니다. -->
                            <input type="radio" id="parentsExist" name="parentsExistence" value="1" onchange="updateHeirInputs()">
                            <label for="parentsExist">생존 (있음)</label>
                            <input type="radio" id="parentsNone" name="parentsExistence" value="0" checked onchange="updateHeirInputs()">
                            <label for="parentsNone">없음</label>
                            <div class="tooltip-container">
                                <span class="text-blue-500 cursor-pointer text-sm">[?]
                                    <div class="tooltip">
                                        **법적 상속 순위:** 직계비속(자녀)이 1명이라도 생존해 있으면 직계존속(부모)은 상속인이 될 수 없습니다. 부모가 선택되어 있더라도, 자녀가 있는 경우 계산에서 제외됩니다. 자녀와 배우자가 모두 없는 경우에만 2순위 상속인이 됩니다.
                                    </div>
                                </span>
                            </div>
                        </div>

                        <!-- 2. 인원수 선택 (생존 선택 시만 표시) -->
                        <div id="parent-count-selection" class="flex items-center space-x-4 pl-8 pt-1 hidden">
                            <label class="text-gray-600 text-sm font-normal">생존 인원수:</label>
                            <input type="radio" id="parentCount1" name="parentCountSelection" value="1" checked onchange="updateHeirInputs()">
                            <label for="parentCount1">1명</label>
                            <input type="radio" id="parentCount2" name="parentCountSelection" value="2" onchange="updateHeirInputs()">
                            <label for="parentCount2">2명</label>
                        </div>
                    </div>
                </div>

                <!-- 형제자매 수 (3순위) -->
                <div id="sibling-section" class="flex items-center space-x-6">
                    <label for="siblingCount" class="block text-gray-700 w-24">형제자매 수</label>
                    <input type="number" id="siblingCount" min="0" value="0"
                           class="w-24 p-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
                           oninput="updateHeirInputs()">
                    <div class="tooltip-container">
                        <span class="text-blue-500 cursor-pointer text-sm">
                            [?]
                            <div class="tooltip">
                                **법적 상속 순위:** 형제자매는 직계비속, 직계존속, 배우자(단독 상속 포함)가 모두 없는 경우에만 3순위 상속인이 될 수 있습니다. 상위 순위 상속인이 있는 경우, 형제자매가 선택되어 있더라도 계산에서 제외됩니다.
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
        // 숫자 콤마 포맷 함수 (천 단위 구분 기호)
        function formatNumber(input) {
            let value = input.value.replace(/,/g, '');
            if (isNaN(value)) {
                input.value = input.value.replace(/[^\d,]/g, '');
                return;
            }
            input.value = new Intl.NumberFormat('ko-KR').format(value);
        }

        // 입력 값(콤마 제거된 숫자) 정리 함수
        function getNumberValue(id) {
            const input = document.getElementById(id).value;
            return parseInt(input.replace(/,/g, '')) || 0;
        }
        
        /**
         * 직계존속(부모) 라디오 버튼에서 선택된 인원수를 반환합니다.
         */
        function getSelectedParentCount() {
            const parentsExist = document.getElementById('parentsExist').checked;
            
            if (!parentsExist) {
                return 0;
            }

            // '생존'이 체크되었을 경우에만 인원수 선택값을 확인
            const countRadios = document.getElementsByName('parentCountSelection');
            for (const radio of countRadios) {
                if (radio.checked) {
                    return parseInt(radio.value);
                }
            }
            return 0; 
        }

        /**
         * 직계존속 인원수 선택 가시성만 업데이트하며, 버튼의 비활성화 로직은 모두 제거합니다.
         */
        function updateHeirInputs() {
            const parentsExistRadio = document.getElementById('parentsExist');
            const parentCountSelectionDiv = document.getElementById('parent-count-selection');

            // '생존 (있음)'이 선택된 경우에만 부모 인원수 선택 창 표시
            if (parentsExistRadio.checked) {
                parentCountSelectionDiv.classList.remove('hidden');
            } else {
                parentCountSelectionDiv.classList.add('hidden');
            }
            
            // NOTE: 형제자매 선택 버튼을 항상 활성화하며, input type="number"로 변경되었으므로 별도의 DOM 조작은 필요 없습니다.
        }

        /**
         * 법정 상속분 계산을 실행하는 함수
         */
        function calculateInheritance() {
            const totalInheritance = getNumberValue('inheritanceAmount');
            if (totalInheritance <= 0) {
                // 커스텀 모달
                const message = document.createElement('div');
                message.innerHTML = '<div class="fixed inset-0 bg-gray-600 bg-opacity-50 flex items-center justify-center z-50"><div class="bg-white p-6 rounded-xl shadow-2xl max-w-sm w-full text-center"><p class="text-xl font-bold text-red-600 mb-4">입력 오류</p><p class="text-gray-700">총 상속액을 정확히 입력해 주세요.</p><button onclick="this.closest(\'.fixed\').remove()" class="mt-4 bg-red-500 text-white px-4 py-2 rounded-lg hover:bg-red-600 transition">닫기</button></div></div>';
                document.body.appendChild(message);
                return;
            }

            const childrenCount = parseInt(document.getElementById('childrenCount').value) || 0;
            const spouseExists = document.getElementById('spouseExists').checked;
            
            const isChildrenPresent = childrenCount > 0;
            
            // 2순위 상속인: 자녀가 없을 때만 부모 선택이 유효합니다.
            const parentCount = isChildrenPresent ? 0 : getSelectedParentCount();
            
            // 3순위 상속인: 1순위, 2순위, 배우자가 모두 없을 때만 형제자매 선택이 유효합니다.
            const isRank1_2_Spouse_Present = isChildrenPresent || parentCount > 0 || spouseExists;
            const siblingCount = isRank1_2_Spouse_Present ? 0 : (parseInt(document.getElementById('siblingCount').value) || 0);

            let units = []; // 상속인별 상속분 단위 (1, 1.5 등)
            let primaryHeirFound = false;

            // 1순위: 직계비속 (자녀)
            if (childrenCount > 0) {
                for (let i = 0; i < childrenCount; i++) {
                    units.push({ type: '직계비속 (자녀)', unit: 1 });
                }
                primaryHeirFound = true;
            }

            // 1순위와 동순위: 배우자 (직계비속이 있을 때)
            if (spouseExists && childrenCount > 0) {
                units.push({ type: '배우자', unit: 1.5 });
                primaryHeirFound = true;
            }

            // 2순위: 직계존속 (부모) 및 배우자 (1순위가 없을 때)
            if (!primaryHeirFound) {
                if (parentCount > 0) { 
                    for (let i = 0; i < parentCount; i++) {
                        units.push({ type: '직계존속 (부모)', unit: 1 });
                    }
                    primaryHeirFound = true; 
                }
                
                // 2순위와 동순위: 배우자 (직계존속만 있을 때)
                if (spouseExists && parentCount > 0) {
                     units.push({ type: '배우자', unit: 1.5 });
                     primaryHeirFound = true;
                }

                // 배우자 단독 상속 (1순위, 2순위 상속인 모두 없을 때)
                if (spouseExists && childrenCount === 0 && parentCount === 0) {
                    units.push({ type: '배우자', unit: 1.5 }); 
                    primaryHeirFound = true;
                }
            }
            
            // 3순위: 형제자매 (1, 2순위 및 배우자가 모두 없을 때)
            if (!primaryHeirFound) {
                if (siblingCount > 0) { 
                    for (let i = 0; i < siblingCount; i++) {
                        units.push({ type: '형제자매', unit: 1 });
                    }
                    primaryHeirFound = true;
                }
            }


            let totalUnits = 0;
            let resultTableBody = '';
            
            if (units.length > 0) {
                totalUnits = units.reduce((sum, heir) => sum + heir.unit, 0);
            } else {
                totalUnits = 0;
                resultTableBody = `<tr><td colspan="3" class="py-3 px-4 text-center text-red-500 font-bold">법정 상속인이 없습니다. 상속 재산은 국가에 귀속될 수 있습니다.</td></tr>`;
            }

            // 개별 상속인 그룹별 합산
            const processedHeirs = {};
            units.forEach(heir => {
                const amount = totalInheritance * (heir.unit / totalUnits); 

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
                const heirOrder = ['배우자', '직계비속 (자녀)', '직계존속 (부모)', '형제자매']; 
                
                heirOrder.forEach(type => {
                    const data = processedHeirs[type];
                    if (!data) return;

                    const unitPerPerson = data.unit / data.count; 
                    const amountPerPerson = data.amount / data.count;
                    const typeDisplay = type.replace('직계비속 (자녀)', '자녀').replace('직계존속 (부모)', '부모'); 

                    // 1. 배우자 (단독 표시)
                    if (type === '배우자') {
                        resultTableBody += `
                            <tr class="hover:bg-green-50">
                                <td class="py-3 px-4">${typeDisplay}</td>
                                <td class="py-3 px-4">
                                    ${data.unit.toFixed(1)}/${totalUnits.toFixed(1)}
                                </td>
                                <td class="py-3 px-4 text-lg font-bold text-gray-800">
                                    ${new Intl.NumberFormat('ko-KR').format(Math.round(data.amount))}원
                                </td>
                            </tr>
                        `;
                    } else {
                        // 2. 기타 상속인 (1인당 및 총액 표시)
                        
                        // 대표 상속인 (1인) 행
                        resultTableBody += `
                            <tr class="hover:bg-green-50">
                                <td class="py-3 px-4">${typeDisplay} (1인당)</td>
                                <td class="py-3 px-4">
                                    ${unitPerPerson.toFixed(1)}/${totalUnits.toFixed(1)}
                                </td>
                                <td class="py-3 px-4 text-base font-semibold text-gray-800">
                                    ${new Intl.NumberFormat('ko-KR').format(Math.round(amountPerPerson))}원
                                </td>
                            </tr>
                        `;

                        // 총 상속분 행 (2명 이상일 때만 표시)
                        if(data.count > 1) {
                             resultTableBody += `
                                <tr class="bg-gray-50 text-gray-600">
                                    <td class="py-2 px-4 italic border-t border-white">${typeDisplay} (총 ${data.count}명)</td>
                                    <td class="py-2 px-4 italic border-t border-white">총 ${data.unit.toFixed(1)}/${totalUnits.toFixed(1)}</td>
                                    <td class="py-2 px-4 italic text-sm border-t border-white">
                                        ${new Intl.NumberFormat('ko-KR').format(Math.round(data.amount))}원 (총액)
                                    </td>
                                </tr>
                            `;
                        }
                    }
                });
            }

            document.getElementById('resultTableBody').innerHTML = resultTableBody;

            // 최종 결과 출력
            document.getElementById('totalAmount').textContent = new Intl.NumberFormat('ko-KR').format(totalInheritance) + '원';
            document.getElementById('totalUnits').textContent = totalUnits.toFixed(1) + '명분';
            document.getElementById('result-area').classList.remove('hidden');
        }

        // 초기 로드 시 및 입력 변경 시 상속인 입력 필드 상태 업데이트
        window.onload = function() {
            updateHeirInputs();
            document.getElementById('childrenCount').value = parseInt(document.getElementById('childrenCount').value) || 0;
            document.getElementById('siblingCount').value = parseInt(document.getElementById('siblingCount').value) || 0; 
            formatNumber(document.getElementById('inheritanceAmount'));
        }
        
        // 상속인 관련 입력 필드에 이벤트 리스너 추가
        document.getElementById('childrenCount').addEventListener('input', updateHeirInputs);
        document.getElementById('spouseExists').addEventListener('change', updateHeirInputs);
        document.getElementById('spouseNone').addEventListener('change', updateHeirInputs);
        
        // 부모 생존 여부 및 인원수 라디오 버튼에 이벤트 리스너 추가
        document.getElementsByName('parentsExistence').forEach(radio => {
            radio.addEventListener('change', updateHeirInputs);
        });
        document.getElementsByName('parentCountSelection').forEach(radio => {
            radio.addEventListener('change', updateHeirInputs);
        });
        
        // 형제자매 수 변경 시 이벤트 리스너 추가
        document.getElementById('siblingCount').addEventListener('input', updateHeirInputs); 
    </script>
</body>
</html>
