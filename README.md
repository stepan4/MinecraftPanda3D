# MinecraftPanda3D

# Hero
# напиши свой код здесь
class Hero:
    def __init__(self, pos, land):
        self.land = land
        self.hero = loader.loadModel('smiley')
        self.hero.setColor(0.5, 0.5, 0.5)
        self.hero.setScale(0.8) 
        self.hero.setPos(pos)
        self.hero.reparentTo(render)
        self.cameraBind()
        self.accept_events()
        self.mode = True


    def cameraBind(self):
        base.disableMouse()
        base.camera.setH(180)
        base.camera.reparentTo(self.hero)   
        base.camera.setPos(0, 0, 1.5) 
        self.cameraOn = True

    def cameraUp(self):
        pos = self.hero.getPos()
        base.mouseInterfaceNode.setPos(-pos[0], -pos[1], -pos[2]-3)
        base.camera.reparentTo(render)
        base.enableMouse()
        self.cameraOn = False


    def accept_events(self):
        base.accept('c', self.change_View)  
        base.accept('c' + '-repeat', self.change_View)

        base.accept('a', self.turn_left)
        base.accept('a' + '-repeat', self.turn_left)

        base.accept('d', self.turn_right)
        base.accept('d' + '-repeat', self.turn_right)

        base.accept('s', self.back)
        base.accept('s' + '-repeat', self.back)

        base.accept('w', self.forward)
        base.accept('w' + '-repeat', self.forward)

        base.accept('r', self.up)
        base.accept('r' + '-repeat', self.up)

        base.accept('f', self.down)
        base.accept('f' + '-repeat', self.down)


        base.accept('b', self.build)
        base.accept('b' + '-repeat', self.build)

        base.accept('v', self.destroy)
        base.accept('v' + '-repeat', self.destroy)

        base.accept('u', self.land.saveMap)
        base.accept('u' + '-repeat', self.land.saveMap)

        base.accept('j', self.land.loadMap)
        base.accept('j' + '-repeat', self.land.loadMap)




        

    def turn_left(self):
        self.hero.setH((self.hero.getH() + 5) %360)

    def turn_right(self):
        self.hero.setH((self.hero.getH() - 5) %360)
    


    def change_View(self):
        if self.cameraOn == True:
            self.cameraUp()
        else:
            self.cameraBind()

    def move_to(self, angle): #angle - Угол поворота
        if self.mode:
            self.just_move(angle)
        else:
            self.try_move(angle)


    def just_move(self, angle):
        pos = self.look_at(angle)
        self.hero.setPos(pos)


    def try_move(self, angle):
        pos = self.look_at(angle)
        if self.land.isEmpty(pos):
            pos = self.land.findHighetsEmpty(pos)
            self.hero.setPos(pos)
        else:
            pos = pos[0], pos[1], pos[2] + 1
            if self.land.isEmpty(pos):
                self.hero.setPos(pos)







    def look_at(self, angle):
 
        x_from = round(self.hero.getX())
        y_from = round(self.hero.getY())
        z_from = round(self.hero.getZ())
 
        dx, dy = self.check_dir(angle)
        x_to = x_from + dx
        y_to = y_from + dy

        return x_to, y_to, z_from


    def check_dir(self, angle):
        if angle >= 0 and angle <= 20:
            return (0, -1)
        elif angle <= 65:
            return (1, -1)
        elif angle <= 110:
            return (1, 0)
        elif angle <= 155:
            return (1, 1)
        elif angle <= 200:
            return (0, 1)
        elif angle <= 245:
            return (-1, 1)
        elif angle <= 290:
            return (-1, 0)
        elif angle <= 335:
            return (-1, -1)
        else:
            return (0, -1)

    


    def back(self):
        angle = (self.hero.getH()+180) % 360
        self.move_to(angle)
    
    def forward(self):
        angle = (self.hero.getH()) % 360
        self.move_to(angle)

    def left(self):
        angle = (self.hero.getH() + 90) % 360
        self.move_to(angle)
 
    def right(self):
        angle = (self.hero.getH() + 270) % 360
        self.move_to(angle)

    def up(self):
        if self.mode:
            self.hero.setZ(self.hero.getZ() + 1)
    
    def down(self):
        if self.mode:
            self.hero.setZ(self.hero.getZ() - 1)


    def build(self):  # Постройка блоков
        angle = self.hero.getH() % 360
        pos = self.look_at(angle)
        if  self.mode:
            self.land.addBlock(pos)
        else:
            self.land.buildBlock(pos)

    def destroy(self): # удаление блоков
        angle = self.hero.getH() % 360
        pos = self.look_at(angle)
        if self.mode:
            self.land.delBlock(pos)
        else:
            self.land.delBlock(pos)


# Mapmanager

from panda3d import *
import pickle
class Mapmanager():
    def __init__(self):
        self.model = 'block.egg'
        self.texture = 'block.png'
        self.colors = [
            (0.2, 0.2, 0.35, 1),
            (0.2, 0.5, 0.2, 1),
            (0.7, 0.2, 0.2, 1),
            (0.5, 0.3, 0.0, 1)
        ]
        self.start_new()
        # self.addBlock((0, 10, 0))

    def start_new(self):
        self.land = render.attachNewNode('Land')

    def getColor(self, z):
        if z < len(self.colors):
            return self.colors[z]
        else:
            return self.colors[len(self.colors) - 1]
    
    def loadLand(self, filename):
        self.clear()
        with open(filename) as file:
            y = 0
            for line in file:
                x = 0
                line = line.split(' ')
                for z in line:
                    for z0 in range(int(z)+1):
                        block = self.addBlock((x, y, z0))
                    x+=1
                y+=1
                         
            

    def addBlock(self,position):
        self.block = loader.loadModel(self.model)
        self.block.setTexture(loader.loadTexture(self.texture))
        self.block.setPos(position)
        self.color = self.getColor(int(position[2]))
        self.block.setColor(self.color)
        self.block.setTag("at", str(position))
        self.block.reparentTo(self.land)

    
    def isEmpty(self, pos): # Для определения есть ли перед нами кубик
        blocks = self.findBlocks(pos)
        if block:
            return False
        else:
            return True

    def findBlocks(self, pos):
        return self.land.findAllMatches('=at=' + str(pos))


    def findHighestEmpty(self, pos): # для определения  верхнего незанятого блока
        x, y, z = pos
        z = 1
        while not self.isEmpty((x, y, z)):
            z += 1
        return (x, y, z)



    def clear(self):
        self.land.removeNode
        self.start_new()


    def delBlock(self, position):
        blocks = self.findBlocks(position)
        for block in blocks:
            block.removeNode()


    def buildBlock(self, pos):
        x, y, z = pos
        pos = x, y, z - 1
        for block in self.findblock(pos):
            block.removeNode()


    def delBlockFrom(self, position):
        x, y, z = self.findHighestEmpty(position)
        pos = x, y, z - 1
        for block in self.findBlocks(pos):
            block.removeNode()



    def saveMap(self):
        blocks = self.land.getChildren()
        with open("my_map.dat", "wb") as fout:
            pickle.dump(len(blocks), fout)

            for block in blocks:
                x, y, z = block.getPos()
                pos = (int(x),int(y) , int(z))
                pickle.dump(pos, fout)


    def loadMap(self):
        self.clear()
        with open("my_map.dat", "rb") as fin :
            length = pickle.load(fin)
            for i in range(length):
                pos = pickle.load(fin)
                self.addBlock(pos)


# GAME

# напиши здесь код основного окна игры
from direct.showbase.ShowBase import ShowBase
from mapmanager import Mapmanager
from hero import Hero
class Game(ShowBase):
    def __init__(self):
        ShowBase.__init__(self)
        self.land = Mapmanager()
        # new_block = Mapmanager()
        base.camLens.setFov(90)
        self.land.loadLand('land.txt')
        self.hero = Hero((10, 10, 5), self.land)
        
game = Game()
game.run()
