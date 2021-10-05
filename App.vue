<template>
  <div>
    <h1>SAFKAS fabelhaftes Vue.JS Jump and Run</h1>
    <h2 v-if = "this.editmode">Editormodus</h2>
    <h2 v-else>Punkte: {{punkte}}</h2>
    <canvas @click=canvasclick ref="Spielcanvas" width=1800 height=700></canvas>
    <br>
    <img :src="sonicsheet" ref="mysonicsheet" style="display:none"/>
    <img :src="sonicsheet2" ref="mysonicsheet2" style="display:none"/>      
    <img :src="monster" ref="mymonster" style="display:none"/>
    <img :src="monster2" ref="mymonster2" style="display:none"/>
    <img :src="muenze" ref="mymuenze" style="display:none"/>
    <img :src="exitschild" ref="myexit" style="display:none"/>
    <img :src="plattform" ref="myplattform" style="display:none"/>
    <button class="nicebutton" @click="Musikclick();">Musik <span v-if = "this.musikgestartet">ausschalten</span><span v-else>einschalten</span></button>
    <button class="nicebutton" @click="NeustartClick();">Neustart</button>
    <button class="nicebutton" @click="ToggleModus();" v-if = "!this.editmode">Leveleditor</button>
    <button class="nicebutton" @click="ToggleModus();" v-else>Spiel starten</button>
    <button class="nicebutton" @click="SetzeWerkzeug(0);" v-if = "this.editmode" v-bind:class="{ aktivesWerkzeug: this.WerkzeugTyp == 0 }">Münzen</button>
    <button class="nicebutton" @click="SetzeWerkzeug(1);" v-if = "this.editmode" v-bind:class="{ aktivesWerkzeug: this.WerkzeugTyp == 1 }">Plattform</button>
    <button class="nicebutton" @click="SetzeWerkzeug(2);" v-if = "this.editmode" v-bind:class="{ aktivesWerkzeug: this.WerkzeugTyp == 2 }">Monster 1</button>
    <button class="nicebutton" @click="SetzeWerkzeug(3);" v-if = "this.editmode" v-bind:class="{ aktivesWerkzeug: this.WerkzeugTyp == 3 }">Monster 2</button>
    <button class="nicebutton" @click="SetzeWerkzeug(4);" v-if = "this.editmode" v-bind:class="{ aktivesWerkzeug: this.WerkzeugTyp == 4 }">Levelausgang</button>
    <button class="nicebutton" @click="SetzeWerkzeug(5);" v-if = "this.editmode" v-bind:class="{ aktivesWerkzeug: this.WerkzeugTyp == 5 }">Löschwerkzeug</button>
    <button class="nicebutton" @click="LevelInhaltClick();" v-show = "this.editmode">Levelinhalt <span v-if = "!this.LevelInhaltAngezeigt">anzeigen</span><span v-else>ausblenden</span></button>
    <pre v-if = "this.editmode && this.LevelInhaltAngezeigt">{{LevelInhalt}}</pre>
  </div>
</template>

<script>
import sonicsheet from "./assets/sonicsheet.png"
import sonicsheet2 from "./assets/sonicsheet2.png"
import monster from "./assets/monster.png"
import monster2 from "./assets/monster2.png"
import muenze from "./assets/Münze.png"
import exitschild from "./assets/Exit.png"
import plattform from "./assets/Plattform.png"

export default {
  data (){    
    return {
      levels: [],
      normalmode: true,
      editmode: false,
      getroffen: false,
      WerkzeugTyp: 0,
      aktlevel: 0,
      maxlevel: 1,
      scrollx: 0,
      scrolly: 0,
      xpos2: 0,
      ypos2: 0,
      punkte: 0,
      LevelInhalt: '',
      gehtnachlinks: false,
      leftpressed: false,
      rightpressed: false,
      toppressed: false,
      musikgestartet: false,
      LevelInhaltAngezeigt: false,
      SpielfigurWeite: 57,
      Spielfigurhoehe: 78,
      ypos: 10,
      xpos: 10,
      speed: 0,
      xspeed: 0,
      laufnr: 0,
      laufnr2: 0,
      laufnrs: 0,
      sonicsheet: sonicsheet,
      sonicsheet2: sonicsheet2,        
      monster: monster,
      monster2: monster2,
      muenze: muenze,
      exitschild: exitschild,
      plattform: plattform,
      audio: ''
    }
  },

  name: 'VueJS-Jump-and-Run',
   
  methods: {
    
    nichtGetroffen: function()
    {
      this.getroffen = false;
    },
    Musikclick: function () {
      this.musikgestartet = !this.musikgestartet;
      if (this.musikgestartet)
      {
        this.audio.play();
      } else
      {
        this.audio.pause();
      }
    },
    canvasclick: function (e) {
      if (this.editmode)
      {
        var xx;
        var yy;
        if (e.pageX || e.pageY) { 
          xx = e.pageX;
          yy = e.pageY;
        }
        else { 
          xx = e.clientX + document.body.scrollLeft + document.documentElement.scrollLeft; 
          yy = e.clientY + document.body.scrollTop + document.documentElement.scrollTop; 
        } 
        xx -= this.$refs['Spielcanvas'].offsetLeft;
        yy -= this.$refs['Spielcanvas'].offsetTop;
        xx+=this.scrollx;
        yy+=this.scrolly;
        // Münze setzen
        if (this.WerkzeugTyp==0)
        {
          this.levels[this.aktlevel].coins.push({ x: xx, y: yy, width: 40, height: 40, collected: false, isgewinn: false});       
        } else
        // Plattform setzen
        if (this.WerkzeugTyp==1)
        {
          this.levels[this.aktlevel].plattformen.push({ x: xx, y:yy, width: 125, height: 20},);       
        } else
        // Monster 1 setzen
        if (this.WerkzeugTyp==2)
        {
          this.levels[this.aktlevel].gegner.push({ xpos: xx, ypos: yy, x: xx, y:yy, width: 64, height: 64, collected: false, speed: 0, typ: 0, istod: false});       
        } else
        // Monster 2 setzen
        if (this.WerkzeugTyp==3)
        {
          this.levels[this.aktlevel].gegner.push({ xpos: xx, ypos: yy, x: xx, y:yy, width: 103, height: 128, collected: false, speed: 0, typ: 1, istod: false});       
        } else
        // Exit setzen
        if (this.WerkzeugTyp==4)
        {
          this.levels[this.aktlevel].coins.push({ x: xx, y: yy, width: 40, height: 40, collected: false, isgewinn: true});       
        } else
        // Levelinhalt löschen
        if (this.WerkzeugTyp==5)
        {
          for (let i=0; i<this.levels[this.aktlevel].plattformen.length;i++)      
          {
            if (xx>this.levels[this.aktlevel].plattformen[i].x && xx < this.levels[this.aktlevel].plattformen[i].x + this.levels[this.aktlevel].plattformen[i].width) {
              if (yy>this.levels[this.aktlevel].plattformen[i].y && yy < this.levels[this.aktlevel].plattformen[i].y + this.levels[this.aktlevel].plattformen[i].height) {
                let t = this.levels[this.aktlevel].plattformen[this.levels[this.aktlevel].plattformen.length-1];
                this.levels[this.aktlevel].plattformen[i]=t;
                this.levels[this.aktlevel].plattformen.pop();                                              
              }
            }
          }
          for (let i=0; i<this.levels[this.aktlevel].gegner.length;i++)      
          {
            if (xx>this.levels[this.aktlevel].gegner[i].x && xx < this.levels[this.aktlevel].gegner[i].x + this.levels[this.aktlevel].gegner[i].width) {
              if (yy>this.levels[this.aktlevel].gegner[i].y && yy < this.levels[this.aktlevel].gegner[i].y + this.levels[this.aktlevel].gegner[i].height) { 
                let t = this.levels[this.aktlevel].gegner[this.levels[this.aktlevel].gegner.length-1];
                this.levels[this.aktlevel].gegner[i]=t;
                this.levels[this.aktlevel].gegner.pop();                                              
              }
            }
          }
          for (let i=0; i<this.levels[this.aktlevel].coins.length;i++)      
          {
            if (xx>this.levels[this.aktlevel].coins[i].x && xx < this.levels[this.aktlevel].coins[i].x + this.levels[this.aktlevel].coins[i].width) {
              if (yy>this.levels[this.aktlevel].coins[i].y && yy < this.levels[this.aktlevel].coins[i].y + this.levels[this.aktlevel].coins[i].height) {
                let t = this.levels[this.aktlevel].coins[this.levels[this.aktlevel].coins.length-1];
                this.levels[this.aktlevel].coins[i]=t;
                this.levels[this.aktlevel].coins.pop();                                              
              }
            }
          }
        }
      }
    },
    canvasmousemove: function (e) {
      if (this.editmode)
      {
        var xx;
        var yy;
        if (e.pageX || e.pageY) { 
          xx = e.pageX;
          yy = e.pageY;
        }
        else { 
          xx = e.clientX + document.body.scrollLeft + document.documentElement.scrollLeft; 
          yy = e.clientY + document.body.scrollTop + document.documentElement.scrollTop; 
        } 
        xx -= this.$refs['Spielcanvas'].offsetLeft;
        yy -= this.$refs['Spielcanvas'].offsetTop;
        xx+=this.scrollx;
        yy+=this.scrolly;
        
        var ctx = this.$refs['Spielcanvas'].getContext('2d');
        if (this.WerkzeugTyp==0)
        {
          ctx.drawImage(this.$refs['mymuenze'], xx, yy);
        } else
        if (this.WerkzeugTyp==1)
        {
          ctx.drawImage(this.$refs['myplattform'], xx, yy);
        } else
        if (this.WerkzeugTyp==2)
        {
          ctx.drawImage(this.$refs['mymonster'], xx, yy);
        } else
        if (this.WerkzeugTyp==3)
        {
          ctx.drawImage(this.$refs['mymonster2'], xx, yy);
        }
        if (this.WerkzeugTyp==4)
        {
          ctx.drawImage(this.$refs['myexit'], xx, yy);
        }
      }
    },
    LevelInhaltClick:  function ()
    {
        if (!this.LevelInhaltAngezeigt)
        {
          this.LevelInhalt = this.levels[this.aktlevel];
        } else
        {
          this.LevelInhalt = '';
        }
        this.LevelInhaltAngezeigt = !this.LevelInhaltAngezeigt;
    },
    ToggleModus:  function () {
      this.editmode = !this.editmode;
      this.neustart();
    },
    SetzeWerkzeug:  function (typ) {
      this.WerkzeugTyp = typ;
    },
    NeustartClick: function () {      
        this.$emit('clicked', true)        
        this.aktlevel = 0;
        this.neustart();
    },
    neustart: function () {
        this.leftpressed = false;
        this.rightpressed = false;
        this.toppressed = false;
        this.getroffen = false;
        this.ypos = 10;
        this.xpos = 10;
        this.punkte = 0;
        for (let i=0; i<this.levels[this.aktlevel].coins.length;i++)      
        {
          this.levels[this.aktlevel].coins[i].collected = false;
        }
        for (let i=0; i<this.levels[this.aktlevel].gegner.length;i++)      
        {
          this.levels[this.aktlevel].gegner[i].x = this.levels[this.aktlevel].gegner[i].xpos;
          this.levels[this.aktlevel].gegner[i].y = this.levels[this.aktlevel].gegner[i].ypos;
          this.levels[this.aktlevel].gegner[i].istod = false;
        }
    },    
    figursteht: function (canhei) {
      if (this.ypos + this.Spielfigurhoehe >= canhei)
      {
        this.ypos = canhei - this.Spielfigurhoehe;
        return true;
      } else {
        for (let i=0; i<this.levels[this.aktlevel].plattformen.length;i++)      
        {
          if (this.xpos+this.SpielfigurWeite > this.levels[this.aktlevel].plattformen[i].x && this.xpos < this.levels[this.aktlevel].plattformen[i].x+ this.levels[this.aktlevel].plattformen[i].width) {
            if (this.ypos+this.Spielfigurhoehe >= this.levels[this.aktlevel].plattformen[i].y && this.ypos+this.Spielfigurhoehe < this.levels[this.aktlevel].plattformen[i].y +this.levels[this.aktlevel].plattformen[i].height)
            {
              this.ypos = this.levels[this.aktlevel].plattformen[i].y - this.Spielfigurhoehe;
              return true;
            }
          }
        }
      }
      return false;
    },        
    feindsteht: function (canhei, index) {
      if (this.levels[this.aktlevel].gegner[index].y + this.levels[this.aktlevel].gegner[index].height > canhei)
      {
        return true;
      } else {
        for (let i=0; i<this.levels[this.aktlevel].plattformen.length;i++)      
        {
          if (this.levels[this.aktlevel].gegner[index].x+this.levels[this.aktlevel].gegner[index].width > this.levels[this.aktlevel].plattformen[i].x && this.levels[this.aktlevel].gegner[index].x < this.levels[this.aktlevel].plattformen[i].x+ this.levels[this.aktlevel].plattformen[i].width) {
            if (this.levels[this.aktlevel].gegner[index].y+this.levels[this.aktlevel].gegner[index].height >= this.levels[this.aktlevel].plattformen[i].y && this.levels[this.aktlevel].gegner[index].y+this.levels[this.aktlevel].gegner[index].height < this.levels[this.aktlevel].plattformen[i].y +this.levels[this.aktlevel].plattformen[i].height)
            {
              return true;
            }
          }
        }
      }
      return false;
    },
    updateCanvas: function (){
      var ctx = this.$refs['Spielcanvas'].getContext('2d');
      ctx.clearRect(0,0,this.$refs['Spielcanvas'].width,this.$refs['Spielcanvas'].height);      

      // <<< Plattformen >>>
      for (let i=0; i<this.levels[this.aktlevel].plattformen.length;i++)      
      {        
        ctx.drawImage(this.$refs['myplattform'], this.levels[this.aktlevel].plattformen[i].x - this.scrollx, this.levels[this.aktlevel].plattformen[i].y- this.scrolly);
      }

      // <<< Gegner >>>
      for (let i=0; i<this.levels[this.aktlevel].gegner.length;i++)      
      {        
        if (this.levels[this.aktlevel].gegner[i].istod) continue;
        // Gegner anzeigen                
        if (this.levels[this.aktlevel].gegner[i].typ==0)
        {
          ctx.drawImage(this.$refs['mymonster'], this.levels[this.aktlevel].gegner[i].x - this.scrollx, this.levels[this.aktlevel].gegner[i].y- this.scrolly);
        }else
        if (this.levels[this.aktlevel].gegner[i].typ==1)
        {
          ctx.drawImage(this.$refs['mymonster2'], this.levels[this.aktlevel].gegner[i].x - this.scrollx, this.levels[this.aktlevel].gegner[i].y- this.scrolly);
        }else {
          ctx.fillRect(this.levels[this.aktlevel].gegner[i].x - this.scrollx, this.levels[this.aktlevel].gegner[i].y- this.scrolly, this.levels[this.aktlevel].gegner[i].width, this.levels[this.aktlevel].gegner[i].height);      
        }

        if (!this.editmode)
        {
        // Gegner links/rechts bewegen
        if (this.levels[this.aktlevel].gegner[i].x + this.levels[this.aktlevel].gegner[i].width/2 > this.xpos+this.SpielfigurWeite/2 + 3 && this.levels[this.aktlevel].gegner[i].x-this.xpos<1000) {
          this.levels[this.aktlevel].gegner[i].x-=3;
        } else
        if (this.levels[this.aktlevel].gegner[i].x + this.levels[this.aktlevel].gegner[i].width/2  < this.xpos+this.SpielfigurWeite/2 -3 && this.xpos-this.levels[this.aktlevel].gegner[i].x<1000 ) {
          this.levels[this.aktlevel].gegner[i].x+=3;
        }
        
        // Das Fallen der Gegner   
        this.levels[this.aktlevel].gegner[i].speed++;         
        let kannspringen = false;
        if (this.levels[this.aktlevel].gegner[i].speed < 0)
        {
          this.levels[this.aktlevel].gegner[i].y+=this.levels[this.aktlevel].gegner[i].speed;
        } else 
        {
          if (this.feindsteht(this.$refs['Spielcanvas'].height,i)) {
            this.levels[this.aktlevel].gegner[i].y-=1;
            if (!this.feindsteht(this.$refs['Spielcanvas'].height,i)) this.levels[this.aktlevel].gegner[i].y+=1;          
          }
          for (let y=0; y< this.levels[this.aktlevel].gegner[i].speed; y++)
          {        
            if (this.feindsteht(this.$refs['Spielcanvas'].height,i)) {              
              this.levels[this.aktlevel].gegner[i].speed = 0;                      
              kannspringen = true;
            } else {
              this.levels[this.aktlevel].gegner[i].y++;
            }
          }
        }
        if (kannspringen && Math.random()<0.05) {
          this.levels[this.aktlevel].gegner[i].speed = -17;
        }
        // Ende: Fallen der Gegner
        }
      }// Ende Gegner

      // << Coins >>
      for (let i=0; i<this.levels[this.aktlevel].coins.length;i++)      
      {
        if (!this.levels[this.aktlevel].coins[i].collected)
        {
        
          if (this.levels[this.aktlevel].coins[i].isgewinn) {
            ctx.drawImage(this.$refs['myexit'], this.levels[this.aktlevel].coins[i].x - this.scrollx, this.levels[this.aktlevel].coins[i].y- this.scrolly);
          } else
          {
            ctx.drawImage(this.$refs['mymuenze'], this.levels[this.aktlevel].coins[i].x - this.scrollx, this.levels[this.aktlevel].coins[i].y- this.scrolly);
          }
        }
      }

      // Coins einsammeln oder Exitschild
      if (!this.editmode)
      {
        for (let i=0; i<this.levels[this.aktlevel].coins.length;i++)
        {
          if (!this.levels[this.aktlevel].coins[i].collected)
          {
            if (this.xpos+this.SpielfigurWeite > this.levels[this.aktlevel].coins[i].x && this.xpos < this.levels[this.aktlevel].coins[i].x+ this.levels[this.aktlevel].coins[i].width)
            {
              if (this.ypos+this.Spielfigurhoehe >= this.levels[this.aktlevel].coins[i].y && this.ypos < this.levels[this.aktlevel].coins[i].y +this.levels[this.aktlevel].coins[i].height)
              {
                // Wenn man ein Exitschild erreicht
                if (this.levels[this.aktlevel].coins[i].isgewinn)
                {
                  if (this.aktlevel<this.maxlevel)
                  {
                    this.aktlevel++;
                  } else {
                    this.aktlevel = 0;
                    alert("Du hast gewonnen!!! Herzlichen Glückwunsch!");
                  }
                  this.neustart();
                } else {
                  // Wenn man eine Münze einsammelt
                  this.levels[this.aktlevel].coins[i].collected = true;
                  this.punkte++;
                }
              }
            }
          }
        }

        // Gegner treffen
        for (let i=0; i<this.levels[this.aktlevel].gegner.length;i++)      
        {
          if (!this.levels[this.aktlevel].gegner[i].istod && this.xpos+this.SpielfigurWeite > this.levels[this.aktlevel].gegner[i].x && this.xpos < this.levels[this.aktlevel].gegner[i].x+ this.levels[this.aktlevel].gegner[i].width)
          {
            if (this.ypos+this.Spielfigurhoehe >= this.levels[this.aktlevel].gegner[i].y && this.ypos < this.levels[this.aktlevel].gegner[i].y +this.levels[this.aktlevel].gegner[i].height)
            {
              if (this.normalmode || this.figursteht(this.$refs['Spielcanvas'].height))
              {
                if (!this.getroffen)
                {
                  this.getroffen = true;
                  if (this.punkte>0)
                  {                    
                    this.punkte = 0;
                    setTimeout(this.nichtGetroffen, 2000);
                  } else {                    
                    setTimeout(this.neustart, 500);            
                  }
                  break;
                }               
              } else {
                this.levels[this.aktlevel].gegner[i].istod=true;
                this.speed = -17;
              }
            }
          }
        }
      }
      
      if (this.xpos < 0) this.xpos = 0;
      if (this.xpos < 300) {
        this.xpos2 = this.xpos;        
        this.scrollx = 0;
      } else
      {
        this.xpos2 = 300;
        this.scrollx = this.xpos - 300;
      }

      if (this.ypos > 200) {
        this.ypos2 = this.ypos;        
        this.scrolly = 0;
      } else
      {
        this.ypos2 = 200;
        this.scrolly = this.ypos - 200;
      }

      // <<< Spielfigur anzeigen >>>
      if (!this.getroffen || Math.random()<0.3)
      {
        // Im freien Fall / Sprung
        if (this.speed<0 || (!this.figursteht(this.$refs['Spielcanvas'].height)))
        {
          if (this.gehtnachlinks)
          {
            let coords= [299,357,420,480,542];
            let ycoords= [405,468];
            switch (this.laufnr)
            {              
              case 0:ctx.drawImage(this.$refs['mysonicsheet2'], coords[0],ycoords[0], (coords[1]-coords[0]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[1]-coords[0]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[1]-coords[0]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 1:ctx.drawImage(this.$refs['mysonicsheet2'], coords[1],ycoords[0], (coords[2]-coords[1]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[2]-coords[1]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[2]-coords[1]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 2:ctx.drawImage(this.$refs['mysonicsheet2'], coords[2],ycoords[0], (coords[3]-coords[2]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[3]-coords[2]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[3]-coords[2]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 3:ctx.drawImage(this.$refs['mysonicsheet2'], coords[3],ycoords[0], (coords[4]-coords[3]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[4]-coords[3]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[4]-coords[3]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
            }             
            this.laufnr2 = (this.laufnr2+1)%2;
            if (this.laufnr2 == 0)
            {
              this.laufnr = this.laufnr-1;
              if (this.laufnr<0)this.laufnr=3;
            }            
          } else
          {  
            let coords= [16,77,135,203,258];
            let ycoords= [405,468];
            switch (this.laufnr)
            {              
              case 0:ctx.drawImage(this.$refs['mysonicsheet'], coords[0],ycoords[0], (coords[1]-coords[0]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[1]-coords[0]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[1]-coords[0]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 1:ctx.drawImage(this.$refs['mysonicsheet'], coords[1],ycoords[0], (coords[2]-coords[1]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[2]-coords[1]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[2]-coords[1]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 2:ctx.drawImage(this.$refs['mysonicsheet'], coords[2],ycoords[0], (coords[3]-coords[2]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[3]-coords[2]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[3]-coords[2]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 3:ctx.drawImage(this.$refs['mysonicsheet'], coords[3],ycoords[0], (coords[4]-coords[3]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[4]-coords[3]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[4]-coords[3]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
            }             
            this.laufnr2 = (this.laufnr2+1)%2;
            if (this.laufnr2 == 0)
            {
              this.laufnr = (this.laufnr+1)%4;
            }            
          }
        } else
        // Wenn die Spielfigur auf festen Boden ist
        if (this.xspeed != 0 || this.speed != 0)
        {
          // laufen
          if (this.gehtnachlinks)
          {
            let coords= [35,105,161,210,271];            
            let ycoords= [145,222];
            switch (this.laufnr)
            {              
              case 0:ctx.drawImage(this.$refs['mysonicsheet2'], coords[0],ycoords[0], (coords[1]-coords[0]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[1]-coords[0]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[1]-coords[0]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 1:ctx.drawImage(this.$refs['mysonicsheet2'], coords[1],ycoords[0], (coords[2]-coords[1]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[2]-coords[1]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[2]-coords[1]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 2:ctx.drawImage(this.$refs['mysonicsheet2'], coords[2],ycoords[0], (coords[3]-coords[2]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[3]-coords[2]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[3]-coords[2]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 3:ctx.drawImage(this.$refs['mysonicsheet2'], coords[3],ycoords[0], (coords[4]-coords[3]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[4]-coords[3]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[4]-coords[3]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
            }             
            this.laufnr2 = (this.laufnr2+1)%2;
            if (this.laufnr2 == 0)
            {
              this.laufnr = this.laufnr-1;
              if (this.laufnr<0)this.laufnr=3;
            }            
          } else
          {  
            let coords= [287,349,397,450,521];
            let ycoords= [145,222];
            switch (this.laufnr)
            {              
              case 0:ctx.drawImage(this.$refs['mysonicsheet'], coords[0],ycoords[0], (coords[1]-coords[0]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[1]-coords[0]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[1]-coords[0]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 1:ctx.drawImage(this.$refs['mysonicsheet'], coords[1],ycoords[0], (coords[2]-coords[1]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[2]-coords[1]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[2]-coords[1]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 2:ctx.drawImage(this.$refs['mysonicsheet'], coords[2],ycoords[0], (coords[3]-coords[2]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[3]-coords[2]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[3]-coords[2]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 3:ctx.drawImage(this.$refs['mysonicsheet'], coords[3],ycoords[0], (coords[4]-coords[3]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[4]-coords[3]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[4]-coords[3]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
            }             
            this.laufnr2 = (this.laufnr2+1)%2;
            if (this.laufnr2 == 0)
            {
              this.laufnr = (this.laufnr+1)%4;
            }            
          }
        } else
        {
          // stehen
          if (this.gehtnachlinks)
          {
            let coords= [102,153,201,252,307,365,429];
            let ycoords= [23,101];
            switch (this.laufnrs)
            {              
              case 0:ctx.drawImage(this.$refs['mysonicsheet2'], coords[0],ycoords[0], (coords[1]-coords[0]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[1]-coords[0]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[1]-coords[0]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 1:ctx.drawImage(this.$refs['mysonicsheet2'], coords[1],ycoords[0], (coords[2]-coords[1]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[2]-coords[1]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[2]-coords[1]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 2:ctx.drawImage(this.$refs['mysonicsheet2'], coords[2],ycoords[0], (coords[3]-coords[2]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[3]-coords[2]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[3]-coords[2]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 3:ctx.drawImage(this.$refs['mysonicsheet2'], coords[3],ycoords[0], (coords[4]-coords[3]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[4]-coords[3]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[4]-coords[3]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 4:ctx.drawImage(this.$refs['mysonicsheet2'], coords[4],ycoords[0], (coords[5]-coords[4]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[5]-coords[4]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[5]-coords[4]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 5:ctx.drawImage(this.$refs['mysonicsheet2'], coords[5],ycoords[0], (coords[6]-coords[5]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[6]-coords[5]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[6]-coords[5]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
            }             
            this.laufnr2 = (this.laufnr2+1)%4;
            if (this.laufnr2 == 0)
            {
              this.laufnrs = this.laufnrs-1;
              if (this.laufnrs<0)this.laufnrs=5;
            }            
          } else
          {
            let coords= [130,187,251,307,355,404,457];
            let ycoords= [23,101];
            switch (this.laufnrs)
            {              
              case 0:ctx.drawImage(this.$refs['mysonicsheet'], coords[0],ycoords[0], (coords[1]-coords[0]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[1]-coords[0]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[1]-coords[0]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 1:ctx.drawImage(this.$refs['mysonicsheet'], coords[1],ycoords[0], (coords[2]-coords[1]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[2]-coords[1]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[2]-coords[1]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 2:ctx.drawImage(this.$refs['mysonicsheet'], coords[2],ycoords[0], (coords[3]-coords[2]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[3]-coords[2]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[3]-coords[2]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 3:ctx.drawImage(this.$refs['mysonicsheet'], coords[3],ycoords[0], (coords[4]-coords[3]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[4]-coords[3]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[4]-coords[3]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 4:ctx.drawImage(this.$refs['mysonicsheet'], coords[4],ycoords[0], (coords[5]-coords[4]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[5]-coords[4]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[5]-coords[4]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
              case 5:ctx.drawImage(this.$refs['mysonicsheet'], coords[5],ycoords[0], (coords[6]-coords[5]),(ycoords[1]-ycoords[0]), this.xpos2, this.ypos2,(coords[6]-coords[5]),(ycoords[1]-ycoords[0]));this.SpielfigurWeite=(coords[6]-coords[5]);this.Spielfigurhoehe=(ycoords[1]-ycoords[0]);break;
            }             
            this.laufnr2 = (this.laufnr2+1)%4;
            if (this.laufnr2 == 0)
            {
              this.laufnrs = (this.laufnrs+1)%6;
            }            
          }
        }
      }
      this.speed++;
      this.xpos += this.xspeed;
      let kannspringen = false;
      // Überprüfen, ob die Spielfigur fällt oder steht und somit springen kann
      if (this.speed < 0)
      {
        this.ypos+=this.speed;
      } else 
      {
        for (let y=0; y < this.speed; y++)
        {        
          if (this.figursteht(this.$refs['Spielcanvas'].height))
          {
            this.speed = 0;          
            kannspringen = true;
            break;
          } else {
            this.ypos++;
          }
        }
      }
      if (kannspringen && this.uppressed) {
        this.speed = -17;
      }
      if (this.leftpressed) {
        if (this.xspeed > -20) this.xspeed -=2;
        this.gehtnachlinks = true;        
      } else {
        if (this.xspeed < 0) {
          this.xspeed +=6;
          if (this.xspeed > 0) this.xspeed = 0;
        }
      }
      if (this.rightpressed) {
        if (this.xspeed < 20) this.xspeed +=2;
        this.gehtnachlinks = false;
      } else {
        if (this.xspeed > 0) 
        {
          this.xspeed -=6;
          if (this.xspeed < 0) this.xspeed = 0;
        }
      }
    },
  },
  created() {
    // Levels manuell definieren
    this.levels = [
      // Level 1
      {
        gegner: [
          { xpos: 400, ypos: 200, x: 0, y:0, width: 64, height: 64, collected: false, speed: 0, typ: 0, istod: false},
          { xpos: 900, ypos: 200, x: 0, y:0, width: 64, height: 64, collected: false, speed: 0, typ: 0, istod: false},
          { xpos: 1400, ypos: 200, x: 0, y:0, width: 103, height: 128, collected: false, speed: 0, typ: 1, istod: false}
        ],
        coins: [
          { x: 100, y:200, width: 40, height: 40, collected: false, isgewinn: false},
          { x: 200, y:240, width: 40, height: 40, collected: false, isgewinn: false},
          { x: 300, y:100, width: 40, height: 40, collected: false, isgewinn: false},
          { x: 800, y:300, width: 40, height: 40, collected: false, isgewinn: false},
          { x: 1800, y:300, width: 64, height: 52, collected: false, isgewinn: true}
        ],
        plattformen: [
          { x: 100, y:-100, width: 125, height: 20},
          { x: 300, y:0, width: 125, height: 20},
          { x: 200, y:100, width: 125, height: 20},
          { x: 0, y:400, width: 125, height: 20},
          { x: 300, y:400, width: 125, height: 20},
          { x: 200, y:300, width: 125, height: 20},
          { x: 400, y:200, width: 125, height: 20},
          { x: 600, y:400, width: 125, height: 20},
          { x: 1000, y:400, width: 125, height: 20},
          { x: 1400, y:400, width: 125, height: 20}
        ]
      },
      // Level 2
      {
        gegner: [
          {xpos:1053,ypos:260,x:1053,y:260,width:64,height:64,collected:false,speed:0,typ:0,istod:false},
          {xpos:2138,ypos:162,x:2138,y:162,width:64,height:64,collected:false,speed:9,typ:0,istod:false},
          {xpos:1734,ypos:95,x:1734,y:95,width:64,height:64,collected:false,speed:0,typ:0,istod:false},
          {xpos:2484,ypos:327,x:2484,y:327,width:103,height:128,collected:false,speed:-15,typ:1,istod:false}
        ],
        coins:  [
          {x:785,y:121,width:40,height:40,collected:false,isgewinn:false},
          {x:525,y:117,width:40,height:40,collected:false,isgewinn:false},
          {x:1985,y:135,width:40,height:40,collected:false,isgewinn:false},
          {x:1681,y:257,width:40,height:40,collected:false,isgewinn:false},
          {x:1809,y:386,width:40,height:40,collected:false,isgewinn:false},
          {x:2037,y:410,width:40,height:40,collected:false,isgewinn:false},
          {x:2713,y:330,width:40,height:40,collected:false,isgewinn:true}
        ],
        plattformen:  [
          {x:100,y:-100,width:125,height:20},{x:95,y:481,width:125,height:20},
          {x:10,y:482,width:125,height:20},{x:220,y:483,width:125,height:20},
          {x:341,y:479,width:125,height:20},{x:462,y:469,width:125,height:20},
          {x:585,y:466,width:125,height:20},{x:702,y:462,width:125,height:20},
          {x:825,y:452,width:125,height:20},{x:941,y:460,width:125,height:20},
          {x:1059,y:475,width:125,height:20},{x:1192,y:490,width:125,height:20},
          {x:1171,y:489,width:125,height:20},{x:1186,y:327,width:125,height:20},
          {x:996,y:356,width:125,height:20},{x:712,y:307,width:125,height:20},
          {x:421,y:338,width:125,height:20},{x:595,y:216,width:125,height:20},
          {x:915,y:192,width:125,height:20},{x:1314,y:479,width:125,height:20},
          {x:1432,y:470,width:125,height:20},{x:1554,y:465,width:125,height:20},
          {x:1675,y:464,width:125,height:20},{x:1799,y:466,width:125,height:20},
          {x:1567,y:325,width:125,height:20},{x:1471,y:171,width:125,height:20},
          {x:1783,y:178,width:125,height:20},{x:1888,y:332,width:125,height:20},
          {x:1917,y:469,width:125,height:20},{x:2037,y:480,width:125,height:20},
          {x:2160,y:485,width:125,height:20},{x:2157,y:343,width:125,height:20},
          {x:2278,y:482,width:125,height:20},{x:2396,y:478,width:125,height:20},
          {x:2516,y:477,width:125,height:20},{x:2630,y:474,width:125,height:20},
          {x:2747,y:475,width:125,height:20},{x:2868,y:476,width:125,height:20},
          {x:2988,y:479,width:125,height:20},{x:3109,y:479,width:125,height:20},
          {x:3005,y:321,width:125,height:20},{x:3225,y:483,width:125,height:20},
          {x:3248,y:369,width:125,height:20}
        ]
      }
    ];
    
    this.audio = new Audio(require('./assets/Musik.mp3'));
    this.audio.autoplay = true;
    this.audio.loop = true;
    this.audio.load();

    window.addEventListener('mousemove', this.canvasmousemove);
    window.addEventListener('keydown', (e) => {
      // << Leveleditor >>
      if (this.editmode)
      {
        // NUM Taste 1 für 1. setzbares Element etc.
        if (e.keyCode == '97') 
        {
          this.WerkzeugTyp = 0
        } else
        if (e.keyCode == '98') 
        {
          this.WerkzeugTyp = 1
        } else
        if (e.keyCode == '99') 
        {
          this.WerkzeugTyp = 2
        } else
        if (e.keyCode == '100') 
        {      
          this.WerkzeugTyp = 3
        } else
        if (e.keyCode == '101') 
        {      
          this.WerkzeugTyp = 4
        } else
        // NUM Taste 6 für Löschen
        if (e.keyCode == '102')
        {      
          this.WerkzeugTyp = 5
        }
      }
      // Taste F1 drücken, um zwischen dem Spiel und den Editor zu wechseln
      if (e.keyCode == '112') 
      {
        this.ToggleModus();
      } else
      // Taste F12 drücken, um zwischen den normalen und den Superman-Modus (Unverwundbarkeit) zu wechseln
      if (e.keyCode == '123')
      {
        this.normalmode = !this.normalmode;        
      } else
      // Pfeiltasten für die Bewegung - gedrückt
      if (e.keyCode == '37') 
      {
        this.leftpressed = true
      } else
      if (e.keyCode == '39') 
      {
        this.rightpressed = true;
      }
      if (e.keyCode == '38') 
      {
        this.uppressed = true;          
      }
    });
    // Pfeiltasten für die Bewegung - losgelassen
    window.addEventListener('keyup', (e) => {
      if (e.keyCode == '37') 
      {
        this.leftpressed = false
      } else
      if (e.keyCode == '39') 
      {
        this.rightpressed = false;
      }
      if (e.keyCode == '38') 
      {
        this.uppressed = false;          
      }
    });

    this.neustart();
  },
  mounted: function (){
      // Setzt die Bildwiederholungsrate
      setInterval(this.updateCanvas, 50);
    }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  width:50%;
  margin: auto;
}
h1 {
  text-align: center;
}
h2 {
  text-align: center;
}
canvas {
  border-width: 1px;
  border-color: #000000;
  border-style: solid;
  margin-left: 10px;
}
.nicebutton {
  margin: 5px;
  background-color: #CCCCCC;
  border-style: solid;
  border-width: 2px;
  border-color: black;
  background-color: #CCFFAA;
  border-radius: 4px;
  padding: 5px;
  -webkit-box-shadow: 10px 10px 24px 0px rgba(0,0,0,0.75);
  -moz-box-shadow: 10px 10px 24px 0px rgba(0,0,0,0.75);
  box-shadow: 10px 10px 24px 0px rgba(0,0,0,0.75);
}
.aktivesWerkzeug {
  background-color: #FFFFFF;
}
</style>