<template>
    <div class="hello">
        <canvas ref="canvas"></canvas>
    </div>
</template>

<script>
let canvas, ctx;
let p1, p2, ball;
const width = 20;
const height = 150;
const digits = [
    "111101101101111", "010010010010010", "111001111100111", "111001111001111", "101101111001001",
    "111100111001111", "111100111101111", "111001001001001", "111101111101111", "111101111001111"
];

export default {
    name: "HelloWorld",
    props: {
        msg: String
    },
    mounted() {
        canvas = this.$refs.canvas;
        ctx = canvas.getContext("2d");

        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
        this.initGame();
        requestAnimationFrame(this.runGame);
        canvas.addEventListener("mousemove", this.updatePlayerMove);
    },
    beforeDestroy() {
        canvas.removeEventListener("mousemove", this.updatePlayerMove);
    },
    methods: {
        initGame() {
            this.initBall();
            this.initPlayer1();
            this.initPlayer2();
        },
        updatePlayerMove(event) {
            const { clientY } = event;
            p1.y = clientY;
        },
        clearBoard() {
            ctx.fillStyle = "#222";
            ctx.fillRect(0, 0, canvas.width, canvas.height);
        },
        drawMiddle() {
            ctx.beginPath();
            ctx.lineWidth = 4;
            ctx.strokeStyle = "#FFF";
            ctx.setLineDash([15, 10]);
            ctx.moveTo(canvas.width / 2, 0);
            ctx.lineTo(canvas.width / 2, canvas.height);
            ctx.stroke();
            ctx.closePath();
        },
        drawScore(score, pos) {
            const digitZoom = 30;
            const drawDigit = (x, y, digit) => {
                digits[digit].split("").forEach((state, idx) => {
                    if (state === "1") {
                        const xAdd = idx % 3;
                        const yAdd = idx / 3 | 0;
                        ctx.fillRect(x + (xAdd * digitZoom), y + (yAdd * digitZoom), digitZoom, digitZoom);
                    }
                });
            };
            const splitNumber = score.toString().split("");
            const digitsCount = splitNumber.length;
            const scoreWidth = (digitsCount * 3 * digitZoom) + (digitsCount - 1) * digitZoom;
            const y = 25;
            const offset = 120;
            const x = (canvas.width / 2 | 0) + (pos === "right" ? offset : (-offset - scoreWidth));

            splitNumber.forEach((digit, idx) => {
                drawDigit(x + (idx * ((3 * digitZoom) + digitZoom)), y, +digit);
            });
        },
        initBall() {
            const angle = (Math.PI / 4) * Math.random();
            ball = {
                x: canvas.width / 2,
                y: canvas.height / 2,
                radius: 15,
                speedX: 6 * Math.cos(angle) * (Math.random() > 0.5 ? -1 : 1),
                speedY: 6 * Math.sin(angle) * (Math.random() > 0.5 ? -1 : 1),
                modifier: 6,
                draw() {
                    ctx.fillStyle = "#FFF";
                    ctx.arc(this.x, this.y, this.radius, 0, 2 * Math.PI);
                    ctx.fill();
                }
            };

            Object.defineProperties(ball, {
                top: {
                    get() {
                        return this.y - this.radius;
                    }
                },
                bottom: {
                    get() {
                        return this.y + this.radius;
                    }
                },
                left: {
                    get() {
                        return this.x - this.radius;
                    }
                },
                right: {
                    get() {
                        return this.x + this.radius;
                    }
                }
            });
        },
        resetBall() {
            Object.assign(ball, {
                x: canvas.width / 2,
                y: canvas.height / 2,
                radius: 15,
                speedX: 5,
                speedY: 5,
                modifier: 6
            });
        },
        initPlayer1() {
            p1 = {
                width,
                height,
                x: width,
                y: canvas.height / 2,
                score: 0,
                draw() {
                    ctx.fillStyle = "#FFF";
                    ctx.fillRect(this.width, this.y - (this.height / 2), -this.width, this.height);
                    ctx.fill();
                }
            };
            Object.defineProperties(p1, {
                top: {
                    get() {
                        return this.y - this.height / 2;
                    }
                },
                bottom: {
                    get() {
                        return this.y + this.height / 2;
                    }
                }
            });
        },
        initPlayer2() {
            p2 = {
                width,
                height,
                x: canvas.width - width,
                y: canvas.height / 2,
                score: 0,
                draw() {
                    ctx.fillStyle = "#FFF";
                    ctx.fillRect(this.x, this.y - (this.height / 2), this.width, this.height);
                    ctx.fill();
                }
            };
            Object.defineProperties(p2, {
                top: {
                    get() {
                        return this.y - this.height / 2;
                    }
                },
                bottom: {
                    get() {
                        return this.y + this.height / 2;
                    }
                }
            });
        },
        checkCollisions() {
            if (ball.top < 0) {
                ball.speedY = -ball.speedY;
                ball.y = ball.radius;
            }

            if (ball.bottom > canvas.height) {
                ball.y = canvas.height - ball.radius;
                ball.speedY = -ball.speedY;
            }

            if (ball.left < 0) {
                p2.score += 1;
                this.resetBall();
            }

            if (ball.right > canvas.width) {
                p1.score += 1;
                this.resetBall();
            }

            if (ball.x < canvas.width / 2) {
                if (ball.left < p1.x && ball.bottom > p1.top && ball.top < p1.bottom) {
                    const yDiff = ball.y - p1.y;
                    const angle = (Math.PI / 4) * (yDiff / (p1.height / 2));
                    ball.speedX = ball.modifier * Math.cos(angle);
                    ball.speedY = ball.modifier * Math.sin(angle);
                    ball.modifier += 0.3;
                }
            } else {
                if (ball.right > p2.x && ball.bottom > p2.top && ball.top < p2.bottom) {
                    const yDiff = ball.y - p2.y;
                    const angle = (Math.PI / 4) * (yDiff / (p2.height / 2));
                    ball.speedX = ball.modifier * Math.cos(angle) * -1;
                    ball.speedY = ball.modifier * Math.sin(angle);
                    ball.modifier += 0.3;
                }
            }
        },
        runGame() {
            this.clearBoard();
            this.drawMiddle();
            [ball, p1, p2].forEach((entity) => entity.draw());

            ball.x += ball.speedX;
            ball.y += ball.speedY;

            p2.y += (ball.y - p2.y) * 0.085;

            this.checkCollisions();

            this.drawScore(p1.score, "left");
            this.drawScore(p2.score, "right");
            requestAnimationFrame(this.runGame);
        }
    }
};
</script>

<style lang="scss">
html, body {
    padding: 0;
    margin: 0;
}
</style>
