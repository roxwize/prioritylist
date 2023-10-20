<script>
    import {Button, Card, CardBody, CardSubtitle, CardTitle, Icon, Tooltip} from "sveltestrap";
    import moment from "moment";
    import {createEventDispatcher} from "svelte";

    const dispatch = createEventDispatcher();

    export let task;
    export let id;
</script>

<Card id="card-{id}" class={task.completed ? "card-complete" : "" + task.priority.label === "Critical" ? "card-critical" : ""}>
    <CardBody style="position:relative;">
        <a href="#" on:click={dispatch("delete", {task:task})}><Icon name="trash3-fill" class="card-close" /></a>
        <CardTitle>
            <Icon id="task-icon-{id}" name={task.priority.icon} class="text-{task.priority.color}" /><strong class="ms-2">{task.name}</strong>
        </CardTitle>
        <Tooltip target="task-icon-{id}" placement="top">{task.priority.label} Priority</Tooltip>
        <CardSubtitle>{task.desc}</CardSubtitle>
        Added {moment(task.addedDate*1000).format("MMMM Do, Y @ hh:mma")}
        {#if task.completionDate}
            <br/>Completed {moment(task.completionDate*1000).format("MMMM Do, Y @ hh:mma")}
        {/if}
        <Button color="primary" class="my-2" style="float:right;" on:click={() => dispatch("mark", {task:task})}>Mark {task.completed ? "incomplete" : "complete"}</Button>
    </CardBody>
</Card>
