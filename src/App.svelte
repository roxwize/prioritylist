<script>
    import {
        Button,
        ButtonGroup,
        ButtonToolbar, Col,
        Container, Dropdown, DropdownItem, DropdownMenu, DropdownToggle,
        Form,
        FormGroup,
        Input,
        InputGroup, InputGroupText, Label, Modal, ModalBody, ModalFooter,
        Offcanvas, Row
    } from "sveltestrap";
    import Task from "./lib/Task.svelte";
    import { v4 as uuid } from "uuid";
    import Cookies from "js-cookie";

    const PRIORITIES = {
        Low: { id: 0, label: "Low", color: "secondary", icon: "exclamation-circle" },
        Medium: { id: 1, label: "Medium", color: "success", icon: "exclamation-circle-fill" },
        High: { id: 2, label: "High", color: "warning", icon: "exclamation-circle-fill" },
        Critical: { id: 3, label: "Critical", color: "danger", icon: "exclamation-diamond-fill" }
    }

    let tasks = {
        complete: {},
        incomplete: {}
    };

    let curName = "";
    let curDesc = "";
    let curPriority = PRIORITIES.Medium;
    let curId = "";
    let curType = "";
    const resetCurTask = () => { curName = ""; curDesc = ""; curPriority = PRIORITIES.Medium; curId = ""; curType = ""; }

    let loaded = false;
    let addMenuOpen = false;
    let addMenuValidated = false;
    let deleteModalOpen = false;
    const openAddMenu = () => addMenuOpen = !addMenuOpen;
    const addTask = () => {
        tasks.incomplete[uuid()] = {
            name: curName,
            desc: curDesc,
            priority: curPriority,
            completed: false
        };
        addMenuOpen = false;
        resetCurTask();
        saveTasks();
    }
    const markTask = (id, task) => {
        if (task.completed) {
            task.completed = false;
            tasks.incomplete[id] = task;
            delete tasks.complete[id];
        } else {
            task.completed = true;
            tasks.complete[id] = task;
            delete tasks.incomplete[id];
        }
        saveTasks();
    }
    const deleteTask = (type, id) => {
        delete tasks[type][id];
        tasks = tasks;
        saveTasks();
    }
    const deleteTaskConfirm = (type, id, task) => {
        curName = task.name;
        curType = type;
        curId = id;
        deleteModalOpen = true;
    }
    // Save tasks to cookie
    const saveTasks = () => {
        Cookies.set("priorityListTasks", btoa(JSON.stringify(tasks)));
    }
    const loadTasks = () => {
        const c = Cookies.get("priorityListTasks");
        if (!c) return true;
        tasks = JSON.parse(atob(Cookies.get("priorityListTasks")));
        return true;
    }

    window.onload = () => {
        loaded = loadTasks();
    }
</script>

<main>
    <div class="m-4">
        <h1>priorityList</h1>
        <Button color="primary" on:click={openAddMenu}>Add a task</Button>
        <hr/>
        {#if !loaded}
            <strong>loading...</strong>
        {/if}
        <Container class="card-container mb-4">
            {#each Object.entries(tasks.incomplete) as [id, task]}
                <Task {id} {task} on:mark={() => markTask(id, task)} on:delete={() => deleteTaskConfirm("incomplete", id, task)} />
            {/each}
        </Container>
        <Container class="card-container">
            {#each Object.entries(tasks.complete) as [id, task]}
                <Task {id} {task} on:mark={() => markTask(id, task)} on:delete={() => deleteTaskConfirm("complete", id, task)} />
            {/each}
        </Container>
    </div>
    <Offcanvas isOpen={addMenuOpen} toggle={openAddMenu} header="Add a task" placement="end">
        <!-- What the fuck -->
        <Form on:submit={addTask}>
            <FormGroup floating label="Name">
                <Input required bind:value={curName}></Input>
            </FormGroup>
            <FormGroup floating label="Description">
                <Input bind:value={curDesc}></Input>
            </FormGroup>
            <Label>Urgency</Label>
            <Dropdown>
                <DropdownToggle caret color={curPriority.color}>{curPriority.label}</DropdownToggle>
                <DropdownMenu>
                    {#each Object.entries(PRIORITIES) as [_, v]}
                        <DropdownItem
                                color={v.color}
                                class={curPriority === v ? "active" : ""}
                                on:click={() => curPriority = v}>{v.label}
                        </DropdownItem>
                    {/each}
                </DropdownMenu>
            </Dropdown>
            <hr/>
            <Button color="primary" on:click={() => { addMenuValidated = true; }}>Create</Button>
        </Form>
    </Offcanvas>
    <Modal isOpen={deleteModalOpen} toggle={() => deleteModalOpen = !deleteModalOpen}>
        <ModalBody>Really delete <strong>{curName}</strong>?</ModalBody>
        <ModalFooter>
            <Button color="danger" on:click={() => {deleteModalOpen = false; deleteTask(curType, curId); resetCurTask();}}>Yes</Button>
            <Button color="success" on:click={() => deleteModalOpen = false}>No</Button>
        </ModalFooter>
    </Modal>
</main>