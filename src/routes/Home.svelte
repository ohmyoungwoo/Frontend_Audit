<script>
    import fastapi from "../lib/api"
    import { link } from 'svelte-spa-router'
    import { page, keyword, is_login } from "../lib/store"
    
    import moment from 'moment/min/moment-with-locales'

    moment.locale('ko')

    let question_list = []
    let size = 10
    let total = 0
    let kw ='' //검색 keyword
    $: total_page = Math.ceil(total/size)  // 스벨트에서 변수앞에 $: 기호를 붙이면 해당 변수는 반응형 변수, 
                                           // total 변수의 값이 API 호출로 인해 그 값이 변하면 total_page 변수의 값도 실시간으로 재 계산된다

    function get_question_list(_page) {
        let params ={
            page: $page,
            size: size,
            keyword: $keyword,
        }
        fastapi('get', '/api/question/list', params, (json) => {
            question_list = json.question_list
            total = json.total
            kw = $keyword
        })
    }

    $: $page, $keyword, get_question_list()    // 함수 앞의 $: 는 $page가 변경되면 함수도 다시 호출하라는 의미임
</script>

<div class="container my-3">
    <div class="row my-3">   
        <div class="col-6">
            <a use:link href="/question-create" 
                class="btn btn-primary {$is_login ? '' : 'disabled'}">진단 등록하기</a>
        </div>
        <div class="col-6">
            <div class="input-group">
                <input type="text" class="form-control" bind:value="{kw}">  <!-- 검색 창 -->
                <button class="btn btn-outline-secondary" on:click={() => {$keyword = kw, $page = 0}}> 
                    찾기
                </button>
            </div>
        </div>
    </div>

    <table class="table">
        <thead>
        <tr class="text-center table-dark">
            <th>번호</th>
            <th style="width:50%">진단 제목</th>
            <th>진단일자</th>
            <th>작성일자</th>
            <th>작성자</th>
            <th>진단보고서</th>
        </tr>
        </thead>
        <tbody>
        {#each question_list as question, i}
        <tr class="text-center">
            <td>{ total - ( $page * size ) - i }</td>
            <td class="text-start">
                <a use:link href="/detail/{question.id}">{question.subject}</a>
            </td>
            <td>{moment(question.audit_date).format("YYYY년 MM월 DD일")}</td>  <!-- 시간표시: hh:mm a : 시간:분 오전/오후 -->
            <td>{moment(question.create_date).format("YYYY년 MM월 DD일")}</td>  <!-- 시간표시: hh:mm a : 시간:분 오전/오후 -->
            <td>{ question.user ? question.user.username : "" }</td>
        </tr>
        {/each}
        </tbody>
    </table>

    <!-- 페이징처리 시작 -->
    <ul class="pagination justify-content-center">
        <!-- 이전페이지 -->
        <li class="page-item {$page <= 0 && 'disabled'}">
            <button class="page-link" on:click="{() => $page--}">이전</button>
        </li>
        <!-- 페이지번호 -->
        {#each Array(total_page) as _, loop_page}
        {#if loop_page >= $page-5 && loop_page <= $page+5} 
        <li class="page-item {loop_page === $page && 'active'}">
            <button on:click="{() => $page = loop_page}" class="page-link">{loop_page+1}</button>
        </li>
        {/if}
        {/each}
        <!-- 다음페이지 -->
        <li class="page-item {$page >= total_page-1 && 'disabled'}">
            <button class="page-link" on:click="{() => $page++}">다음</button>
        </li>
    </ul>
    <!-- 페이징처리 끝 -->
</div>