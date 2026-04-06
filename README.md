<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Twinkle Drop Lite</title>

<style>
body{background:#111;color:#fff;text-align:center;font-family:sans-serif;}
canvas{background:#fff;margin-top:10px;border-radius:10px;}
button{font-size:18px;padding:10px;margin:5px;border:none;border-radius:8px;background:#00c853;color:#fff;}
</style>
</head>

<body>

<h2>Twinkle Drop</h2>
<p id="info"></p>

<canvas id="game" width="300" height="300"></canvas><br>
<button onclick="drop()">DROP</button>
<button onclick="doubleUp()">DOUBLE</button>

<script>
const canvas=document.getElementById("game");
const ctx=canvas.getContext("2d");

const SIZE=100;
const SYMBOLS=["🍒","🔔","🍀","💎","⭐"];
const VALUES={"🍒":10,"🔔":20,"🍀":30,"💎":50,"⭐":80};

let grid=[[null,null,null],[null,null,null],[null,null,null]];
let ice=[[false,false,false],[false,false,false],[false,false,false]];

let medals=100;
let free=0;
let lastWin=0;

let order=[[0,0],[1,0],[2,0],[2,1],[2,2],[1,2],[0,2],[0,1],[1,1]];
let step=0;

function draw(){
  ctx.clearRect(0,0,300,300);

  for(let y=0;y<3;y++){
    for(let x=0;x<3;x++){
      ctx.strokeRect(x*SIZE,y*SIZE,SIZE,SIZE);

      let cell=grid[y][x];
      if(cell){
        ctx.font="40px sans-serif";
        ctx.fillText(cell.type,x*SIZE+30,y*SIZE+60);
      }

      if(ice[y][x]){
        ctx.fillStyle="rgba(0,200,255,0.3)";
        ctx.fillRect(x*SIZE,y*SIZE,SIZE,SIZE);
        ctx.fillStyle="#000";
      }
    }
  }

  document.getElementById("info").innerText=
  "Medals:"+medals+" Free:"+free+" LastWin:"+lastWin;
}

function newSymbol(forceWild=false){
  if(forceWild) return {type:"W",life:5};
  if(Math.random()<0.1) return {type:"W",life:5};
  return {type:SYMBOLS[Math.floor(Math.random()*SYMBOLS.length)]};
}

function drop(){
  if(step>=9) return;

  let [x,y]=order[step];

  let sym=newSymbol(free>0); // フリー中は必ずWILD

  grid[y][x]=sym;

  if(Math.random()<0.3) ice[y][x]=true;

  step++;

  if(step>=9){
    resolve();
    step=0;
  }

  draw();
}

function resolve(){
  let combo=0;
  let totalWin=0;

  while(true){
    let removed=false;
    let visited={};

    for(let y=0;y<3;y++){
      for(let x=0;x<3;x++){
        let cell=grid[y][x];
        if(!cell||visited[y+"-"+x])continue;

        let type=cell.type;
        let stack=[[x,y]];
        let group=[];
        visited[y+"-"+x]=true;

        while(stack.length){
          let [cx,cy]=stack.pop();
          group.push([cx,cy]);

          [[1,0],[-1,0],[0,1],[0,-1]].forEach(([dx,dy])=>{
            let nx=cx+dx,ny=cy+dy;
            if(nx>=0&&nx<3&&ny>=0&&ny<3){
              let ncell=grid[ny][nx];
              if(!visited[ny+"-"+nx] && ncell &&
                 (ncell.type==type || ncell.type=="W" || type=="W")){
                visited[ny+"-"+nx]=true;
                stack.push([nx,ny]);
              }
            }
          });
        }

        if(group.length>=3){
          combo++;
          removed=true;

          let base=VALUES[type]||20;
          totalWin += base * combo;

          group.forEach(([gx,gy])=>{
            if(!ice[gy][gx]){
              let c=grid[gy][gx];
              if(c.type=="W"){
                c.life--;
                if(c.life<=0) grid[gy][gx]=null;
              }else{
                grid[gy][gx]=null;
              }
            }else{
              ice[gy][gx]=false;
            }
          });
        }
      }
    }

    if(!removed) break;

    // 落下
    for(let x=0;x<3;x++){
      for(let y=2;y>=0;y--){
        if(!grid[y][x]){
          for(let k=y-1;k>=0;k--){
            if(grid[k][x]){
              grid[y][x]=grid[k][x];
              grid[k][x]=null;
              break;
            }
          }
        }
      }
    }
  }

  if(combo>=4) totalWin*=2; // コンボボーナス

  medals+=totalWin;
  lastWin=totalWin;

  // フリーゲーム突入（適当条件）
  if(Math.random()<0.2){
    free+=3;
  }

  if(free>0) free--;

  // リセット
  grid=[[null,null,null],[null,null,null],[null,null,null]];
  ice=[[false,false,false],[false,false,false],[false,false,false]];
}

function doubleUp(){
  if(lastWin<=0) return;

  let player=Math.random();
  let dealer=Math.random();

  if(player>dealer){
    medals+=lastWin;
    lastWin*=2;
  }else{
    lastWin=0;
  }

  draw();
}

draw();
</script>

</body>
</html>
