<script>
    import { push } from 'svelte-spa-router'
    import fastapi from "../lib/api"
    import Error from "../components/Error.svelte"

    let error = {detail:[]}
    let subject = ''
    let content = ''
    let myBindDate = (new Date()).toJSON().slice(0, 10);
    let date1 = "2024-01-01"; //audit_date test용

    function post_question(event) {
        event.preventDefault()
        let url = "/api/question/create"
        let params = {
            subject: subject,
            content: content,
            audit_date:myBindDate,
        }
        fastapi('post', url, params, 
            (json) => {
                push("/")
            },
            (json_error) => {
                error = json_error
            }
        )
    }
</script>

<div class="container">
    <h5 class="my-3 border-bottom pb-2">진단결과 등록</h5>
    <Error error={error} />
    <form method="post" class="my-3">
        <div class="mb-3">
            <label for="subject">제목</label>
            <input type="text" class="form-control" bind:value="{subject}">
        </div>
        <div class="mb-3">
            <label for="content">내용</label>
            <textarea class="form-control" rows="10" bind:value="{content}"></textarea>
        </div>
        <div class="my-3">
            <label for="audit_date">진단일자</label>
            <input type="date" class="form-control" bind:value="{myBindDate}">
        </div>
        <button class="btn btn-primary" on:click="{post_question}">저장하기</button>
    </form>
</div>