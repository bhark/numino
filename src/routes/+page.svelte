<script>

    import { browser } from '$app/environment'
    import { onMount } from 'svelte'

    let windowHeight
    let windowWidth

    const circleData = {
        pointSize: 4,
        fontSize: '20px',
        padding: 20
    }

    $: state = {
        modulus: 9,
        multiplier: 2,
        points: []
    }
    
    function calculatePosition (angle) {
        return {
            x: Math.cos(((angle) - 90) * (Math.PI / 180)) * (circleData.radius - circleData.padding) + circleData.radius,
            y: Math.sin(((angle) - 90) * (Math.PI / 180)) * (circleData.radius - circleData.padding) + circleData.radius
        }
    }

    const populateState = () => {
        let points = []
        const baseline = 360 / state.modulus

        for (let i = 0; i < state.modulus; i++) {
            const { x, y } = calculatePosition(baseline * i)
            const label = i === 0 ? state.modulus : i
            const angle = baseline * i

            let connection = getDigitalRoot((label * state.multiplier) % state.modulus)
            
            // const digitalRoot = getDigitalRoot(label * state.multiplier)
            // const connection = !digitalRoot 
            //     || digitalRoot === label 
            //     ? null 
            //     : digitalRoot

            points.push({
                label,
                angle,
                connection,
                position: { x, y } 
            })
        }

        state.points = points
    }

    function getDigitalRoot(number) {
        let sum = number
        let arr = []
        
        const reducer = (a,b) => parseInt(a) + parseInt(b)

        while (sum > state.modulus) {
            arr = sum.toString().split('')
            sum = arr.reduce(reducer)
        }

        return sum
    }   

    const drawCircle = (context) => {
        context.beginPath()
        context.arc(
            circleData.radius, 
            circleData.radius, 
            circleData.radius - circleData.padding,
            0,
            2 * Math.PI,
            false
        )
        context.strokeStyle = '#fff'
        context.stroke()
    }

    const drawPoint = (context, point) => {
        context.beginPath()
        context.arc(
            point.position.x,
            point.position.y,
            2,
            0,
            2 * Math.PI
        )
        context.fillStyle = '#fff'
        context.fill()
    }

    const drawLine = (context, startAngle, endAngle) => {
        context.beginPath()
        const { x: startX, y: startY } = calculatePosition(startAngle)
        const { x: endX, y: endY } = calculatePosition(endAngle)
        const distance = Math.hypot((startX - endX), (startY - endY))
        const distancePercentage = (distance / (circleData.radius * 2)) * 100
        const color = (360 * distancePercentage) / 100
        console.log(`${Math.floor(distance)} becomes ${Math.floor(color)}`)
        context.moveTo(startX, startY)
        context.lineTo(endX, endY)
        context.strokeStyle = `hsl(${color}, 90%, 50%)`
        context.stroke()
    }
    
    const render = () => {

        if (state.modulus <= 1)
            state.modulus = 1

        if (state.multiplier <= 1)
            state.multiplier = 1
        
        // get reference to canvas
        const context = document.querySelector('canvas').getContext('2d')

        // clean up
        context.clearRect(0, 0, circleData.canvasWidth, circleData.canvasHeight)

        // draw a circle
        drawCircle(context)

        // calculate state values (points and lines, their positions etc.)
        populateState()

        state.points.forEach((e) => {
            drawPoint(context, e) // draw the point
            if (e.connection !== null && state.points[e.connection]) { // if this point is connected, draw a line to its connection
                drawLine(context, e.angle, state.points[e.connection].angle)
            }
        })
    }

    onMount(async() => {
        const context = 
            document.querySelector('canvas')
            .getContext('2d')

        const availableSpace = Math.min(windowHeight, windowWidth)
        circleData.canvasWidth = circleData.canvasHeight = context.canvas.width = context.canvas.height = availableSpace / 1.3
        circleData.radius = circleData.canvasWidth / 2

        render()
    })

    let handlesShown = true


</script>

<svelte:window 
    bind:innerHeight={windowHeight} 
    bind:innerWidth={windowWidth}
    on:keydown={(e) => {
        if (e.key === ' ' || e.code === 'Space') {
            state.modulus = Math.floor(Math.random() * 1000)
            state.multiplier = Math.floor(Math.random() * 1000)
            render()
        }
    }} />

<div class="main">

    <div class="handles" class:shown={handlesShown}>
        <span class="hide" on:click={() => handlesShown = !handlesShown}>
            <svg width="24px" height="24px" stroke-width="1.5" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" color="#000000"><path d="M6.758 17.243L12.001 12m5.243-5.243L12 12m0 0L6.758 6.757M12.001 12l5.243 5.243" stroke="#000000" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"></path></svg>
        </span>

        <div class="inputs">
            <div class="input-group">
                <span class="label">Modulo</span>
                <input type="range" bind:value={state.modulus} on:input={render} min="1" max="1000">
                <div class="value-container">
                    <button class="subtract" on:click={() => (state.modulus -= 1, render())}>
                        <svg width="24px" height="24px" stroke-width="1.5" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" color="currentColor"><path d="M6 12h12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"></path></svg>
                    </button>
                    <span class="value" contenteditable bind:innerHTML={state.modulus} on:input={render}></span>
                    <button class="add" on:click={() => (state.modulus += 1, render())}>
                        <svg width="24px" height="24px" stroke-width="1.5" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" color="currentColor"><path d="M6 12h6m6 0h-6m0 0V6m0 6v6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"></path></svg>
                    </button>
                </div>
            </div>
            <div class="input-group">
                <span class="label">Multiplikator</span>
                <input type="range" placeholder="Modulus" bind:value={state.multiplier} on:input={render} min="1" max="1000">
                <div class="value-container">
                    <button class="subtract" on:click={() => (state.multiplier -= 1, render())}>
                        <svg width="24px" height="24px" stroke-width="1.5" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" color="currentColor"><path d="M6 12h12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"></path></svg>
                    </button>
                    <span class="value" contenteditable bind:innerHTML={state.multiplier}></span>
                    <button class="add" on:click={() => (state.multiplier += 1, render())}>
                        <svg width="24px" height="24px" stroke-width="1.5" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" color="currentColor"><path d="M6 12h6m6 0h-6m0 0V6m0 6v6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"></path></svg>
                    </button>
                </div>
            </div>
        </div>

        <button on:click={() => {
            state.modulus = Math.floor(Math.random() * 1000)
            state.multiplier = Math.floor(Math.random() * 1000)
            render()
        }}>Tilfældig</button>
    </div>

    <canvas on:click={() => {
        state.modulus = Math.floor(Math.random() * 1000)
        state.multiplier = Math.floor(Math.random() * 1000)
        render()
    }} id="canvas" class:handles-hidden={!handlesShown} />

    <span class="tm">
        <span class="text">Til Far, fra din Søn</span>
        <span class="icon">
            <svg width="18px" height="18px" stroke-width="1.5" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" color="#fff"><path d="M22 8.862a5.95 5.95 0 01-1.654 4.13c-2.441 2.531-4.809 5.17-7.34 7.608-.581.55-1.502.53-2.057-.045l-7.295-7.562c-2.205-2.286-2.205-5.976 0-8.261a5.58 5.58 0 018.08 0l.266.274.265-.274A5.612 5.612 0 0116.305 3c1.52 0 2.973.624 4.04 1.732A5.95 5.95 0 0122 8.862z" stroke="#fff" stroke-width="1.5" stroke-linejoin="round"></path></svg>
        </span>
    </span>

</div>

<style lang="scss">

    .main {
        height: 100vh;
        width: 100vw;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        background-color: #000;
    }

    .tm {
        position: fixed;
        bottom: 20px;
        left: 50%;
        z-index: 2;
        color: #fff;
        font-family: monospace;
        transform: translateX(-50%);
        display: flex;
        flex-direction: column;
        text-transform: uppercase;
        justify-content: center;
        align-items: center;
        gap: 10px;
    }

    canvas {
        overflow: visible;
    }

    .handles {
        background-color: #fff;
        position: fixed;
        top: 20px;
        left: 20px;
        padding: 20px;
        border-radius: 20px;
        display: flex;
        flex-direction: column;
        transition: ease all 500ms;

        &:not(.shown) {
            transform: translateX(-100%);
            background-color: #000;

            .hide {
                svg { transform: rotate(45deg); }
            }
        }

        .hide {
            cursor: pointer;
            width: 30px;
            height: 30px;
            position: absolute;
            top: 0px;
            right: 0%;
            background-color: #fff;
            transform: translate(calc(-50% + 30px), -50%);
            display: flex;
            justify-content: center;
            align-items: center;
            border-radius: 50%;
            box-shadow: rgba(0, 0, 0, 0.24) 0px 3px 8px;
            transition: ease all 500ms;

            svg { transition: ease all 500ms; }
        }

        button {
            background-color: #000;
            padding: 10px;
            color: #fff;
            border: solid 2px #000;
            margin-top: 20px;
            border-radius: 10px;
            cursor: pointer;
            text-transform: uppercase;
            font-family: sans-serif;
            font-weight: bold;
            transition: ease all 300ms;

            &:hover {
                background-color: #fff;
                color: #000;
                border-color: #000;
            }
        }

        .inputs {
            display: flex;
            flex-direction: row;
            justify-content: center;
            align-items: center;
        }

        .input-group {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-family: monospace;
            gap: 10px;

            .value-container {
                display: flex;
                flex-direction: row;
                justify-content: center;
                align-items: center;
                gap: 10px;

                button {
                    width: 30px;
                    height: 30px;
                    display: flex;
                    justify-content: center;
                    align-items: center;
                    padding: 2px;
                    border-radius: 50%;
                    margin-top: 10px;
                }
            }

            span {
                font-size: 1.2rem;
                text-transform: uppercase;

                &.value {
                    background-color: #000;
                    color: #fff;
                    width: 50px;
                    height: calc(50px - 6px);
                    border-radius: 50%;
                    display: flex;
                    justify-content: center;
                    align-items: center;
                    padding-top: 6px;
                    margin-top: 10px;
                }
            }
        }
    }

    @media (max-width: 1400px) {
        div.handles {
            padding: 10px;
            
            .inputs .input-group {
                span {
                    font-size: 14px;

                    &.value {
                        width: 30px;
                        height: calc(30px - 6px);
                        padding-top: 6px;
                    }
                }
            }
        }
    }

    @media (max-width: 600px) {
        canvas {
            transform: translateY(20%);
            transition: ease all 500ms;

            &.handles-hidden {
                transform: translateY(-5%);
            }
        }
        div.handles {
            padding: 10px;
            width: calc(100vw - 60px);
            margin-left: 20px;
            top: 0px;
            left: 0px;
            border-radius: 0 0 20px 20px;
            padding-top: 20px;
            z-index: 10;

            .hide {
                transform: translate(-20%, 20%);
            }

            &:not(.shown) {
                transform: none;
                background-color: transparent;

                .input-group, button {
                    opacity: 0;
                    pointer-events: none;
                }

                .hide {
                    right: 50%;
                    transform: translate(50%, 50%);
                }
            }

            button {
                margin-top: 10px;
                padding: 10px;
                font-size: .8rem;
            }

            .inputs {
                flex-direction: column;
            }
            
            .inputs .input-group {
                margin-bottom: 20px;

                .value-container {
                    display: none;
                }

                span {
                    font-size: 14px;
                }
            }
        }
    }

</style>