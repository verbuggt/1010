##1010
A Python Version of the mobile-game 1010
```python

from tkinter import *
from random import *


class Main:
    def __init__(self):

        self.window = Tk()
        self.window.title("1010")
        self.window.geometry("600x750")
        self.window.configure(background='#474747')
        self.window.resizable(False, False)

        self.game = Game(self)

        self.last_x = None
        self.last_y = None
        self.last_preview = []

        self.points_label = Label(self.window, font=("Segoe UI Light", 24), bg="#474747", fg="lightgray")
        self.points_label["text"] = "0"
        self.points_label.place(x=(300 - self.points_label.winfo_width() / 2), y=10)

        self.canvas = Canvas(self.window, width=500, height=500, bg="lightgray", highlightthickness=0)
        self.canvas.bind("<Button-1>", self.canvas_click)
        self.canvas.bind("<Motion>", self.render_preview)
        self.canvas.bind("<Leave>", self.remove_last_values)
        self.canvas.place(x=50, y=75)

        self.lose_img = PhotoImage(file='./resources/LoseScreenOverlay.gif')
        self.img = PhotoImage(file='./resources/DragAndDropOverlay.gif')
        self.bc_overlay = PhotoImage(file='./resources/BlockCanvasOverlay.gif')

        self.block_canvas = Canvas(self.window, width=500, height=125, bg="lightgray", highlightthickness=0)
        self.block_canvas.place(x=50, y=525 + 50 + 25)

        self.block_canvas.create_image(0, 0, image=self.bc_overlay, anchor="nw")
        self.img_id = self.canvas.create_image(0, 0, image=self.img, anchor="nw")

        self.game.generate_blocks()
        self.render_current_blocks()

        # GUILoseScreen(self.window, self.game, self.lose_img)

        self.window.mainloop()

    def canvas_click(self, event):
        x = int(event.x / 50)
        y = int(event.y / 50)
        if (x < 10) and (y < 10):
            if self.game.selected_block is not None:
                coordinates = self.game.selected_block.coord_array
                if self.game.fits(x, y, coordinates):
                    self.place(x, y, coordinates)
                    block = self.game.selected_block
                    block.destroy()
                    self.game.selected_block = None
                    self.game.current_blocks.remove(block)
                    if len(self.game.current_blocks) == 0:
                        self.game.generate_blocks()
                        self.render_current_blocks()

            if len(self.game.check_lines()) > 0:
                for lines in self.game.check_lines():
                    self.game.clear_line(lines)
                    for i in range(0, 10):
                        self.clear_rect_on_coordinates(i, lines)

            if len(self.game.check_columns()) > 0:
                for columns in self.game.check_columns():
                    self.game.clear_column(columns)
                    for i in range(0, 10):
                        self.clear_rect_on_coordinates(columns, i)

            if not self.game.is_action_possible():
                GUILoseScreen(self.window, self.game, self.lose_img)

    def render_preview(self, event):
        x = int(event.x / 50)
        y = int(event.y / 50)
        if self.last_x != x or self.last_y != y:
            self.last_x = x
            self.last_y = y
            if self.game.selected_block is not None:
                if 0 <= x and 0 <= y and x < 10 and y < 10:
                    if self.game.fits(x, y, self.game.selected_block.coord_array):
                        for index in range(0, len(self.last_preview)):
                            lx = self.last_preview[index][0]
                            ly = self.last_preview[index][1]
                            if self.game.field[ly][lx] == 0:
                                self.draw_rect(self.last_preview[index][0], self.last_preview[index][1], "lightgray")
                        if self.game.selected_block is not None:
                            ca = self.game.selected_block.coord_array
                            self.last_preview = []
                            for index in range(0, len(ca)):
                                tx = x + ca[index][0]
                                ty = y + ca[index][1]
                                if tx < 10 and ty < 10:
                                    self.draw_rect(tx, ty, "yellow")
                                    self.last_preview.append([x + ca[index][0], y + ca[index][1]])

    def place(self, x, y, coordinates):
        for index in range(0, len(coordinates)):
            self.draw_rect_on_coordinates(x + coordinates[index][0], y + coordinates[index][1])
            self.game.set_filed(x + coordinates[index][0], y + coordinates[index][1], 1)

    def remove_last_values(self, event):
        self.last_x = None
        self.last_y = None
        for index in range(0, len(self.last_preview)):
            lx = self.last_preview[index][0]
            ly = self.last_preview[index][1]
            if self.game.field[ly][lx] == 0:
                self.draw_rect(self.last_preview[index][0], self.last_preview[index][1], "lightgray")

    def draw_rect_on_coordinates(self, x, y):
        self.draw_rect(x, y, "orange")

    def clear_rect_on_coordinates(self, x, y):
        self.draw_rect(x, y, "lightgray")

    def draw_rect(self, x, y, color):
        x = x * 50
        y = y * 50
        self.canvas.create_rectangle(x, y, x + 50, y + 50, fill=color, outline="")
        self.restore_grid(self.img_id)

    def render_current_blocks(self):
        for index in range(0, len(self.game.current_blocks)):
            c = self.game.current_blocks[index].get_block_canvas()
            c.place(x=50 + 166 * (index + 1) - 83 - int(c["width"]) / 2, y=75 + 500 + 25 + (62 - int(c["height"]) / 2))

    def restore_grid(self, img_id):
        self.img_id = self.canvas.create_image(0, 0, image=self.img, anchor="nw")
        self.canvas.delete(img_id)


class GUILoseScreen:
    def __init__(self, window, game, lose_img):
        canvas = Canvas(window, width=600, height=725, bg="#474747", highlightthickness=0)
        canvas.create_image(0, 0, image=lose_img, anchor="nw")
        canvas.place(x=0, y=0)


class Game:
    def __init__(self, gui):
        self.gui = gui
        self.field = [
            [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]]

        self.points = 0
        self.blocks = BLOCKS()
        self.current_blocks = []
        self.selected_block = None

    def check_lines(self):
        lines = []
        for line in range(0, 10):
            flag = 1
            for i in range(0, 10):
                if self.field[line][i] != 1:
                    flag = 0
                    break
            if flag == 1:
                lines.append(line)
        return lines

    def check_columns(self):
        columns = []
        for column in range(0, 10):
            flag = 1
            for i in range(0, 10):
                if self.field[i][column] != 1:
                    flag = 0
                    break
            if flag == 1:
                columns.append(column)
        return columns

    def get_points(self):
        return self.points

    def add_points(self, points):
        self.points += points
        self.gui.points_label["text"] = str(self.points)
        self.gui.points_label.place(x=(300 - self.gui.points_label.winfo_width() / 2), y=10)

    def clear_line(self, index):
        for i in range(0, 10):
            self.set_filed(i, index, 0)

    def clear_column(self, index):
        for i in range(0, 10):
            self.set_filed(index, i, 0)

    def set_filed(self, x, y, full):
        self.add_points(1)
        self.field[y][x] = full

    def generate_blocks(self):
        self.current_blocks = []
        for i in range(0, 3):
            self.current_blocks.append(Block(randint(0, len(self.blocks.block_list) - 1), self.blocks, self.gui))

    def fits(self, x, y, coordinates):
        for index in range(0, len(coordinates)):
            tx = x + coordinates[index][0]
            ty = y + coordinates[index][1]

            if 0 <= tx < 10 and 0 <= ty < 10:
                if self.field[ty][tx] == 1:
                    return False
            else:
                return False
        return True

    def is_action_possible(self):
        for y in range(0, len(self.field)):
            for x in range(0, len(self.field[y])):
                for block in self.current_blocks:
                    if self.fits(x, y, block.coord_array):
                        return True
        return False


class Block:
    def __init__(self, block_list_index, blocks, gui):
        self.block_list_index = block_list_index
        self.coord_array = blocks.block_list[block_list_index]
        self.gui = gui
        self.window = gui.window
        self.height = 0
        self.width = 0
        self.width_neg = 0
        self.set_measurement()
        self.canvas = self.__create_block_canvas()

    def set_measurement(self):
        width_pos = 0
        width_neg = 0
        height = 0
        for index in range(0, len(self.coord_array)):
            x1 = self.coord_array[index][0] * 25
            y1 = self.coord_array[index][1] * 25

            if x1 >= 0:
                if x1 + 25 > width_pos:
                    width_pos = x1 + 25
            elif x1 * -1 > width_neg:
                width_neg = (x1 * -1)

            if y1 + 25 > height:
                height = y1 + 25
        self.height = height
        self.width = width_pos + width_neg
        self.width_neg = width_neg

    def get_block_canvas(self):
        return self.canvas

    def __create_block_canvas(self):
        canvas = Canvas(self.window, width=self.width, height=self.height, bg="lightgray", highlightthickness=0)
        canvas.bind("<Button-1>", self.select_block)
        for index in range(0, len(self.coord_array)):
            x1 = self.coord_array[index][0] * 25
            y1 = self.coord_array[index][1] * 25
            canvas.create_rectangle(x1 + self.width_neg, y1, x1 + 25 + self.width_neg, y1 + 25, fill="orange",
                                    outline="")

        return canvas

    def select_block(self, event):
        selected_block = self.gui.game.selected_block
        if selected_block is not None and selected_block is not self:
            selected_block.remove_outline()
        self.gui.game.selected_block = self
        self.canvas["highlightthickness"] = 1

    def remove_outline(self):
        self.canvas["highlightthickness"] = 0

    def destroy(self):
        self.canvas.destroy()


class BLOCKS: # enum
    def __init__(self):
        self.block_list = [
            [[0, 0]],
            [[0, 0], [0, 1]],
            [[0, 0], [0, 1], [0, 2]],
            [[0, 0], [0, 1], [0, 2], [0, 3]],
            [[0, 0], [0, 1], [1, 0]],
            [[0, 0], [1, 0], [2, 0]],
            [[0, 0], [1, 0], [2, 0], [3, 0]],
            [[0, 0], [0, 1], [1, 1]],
            [[0, 0], [1, 0], [1, 1]],
            [[0, 0], [0, 1], [-1, 1]],
            [[0, 0], [0, 1], [1, 0], [1, 1]],
            [[0, 0], [1, 0], [2, 0], [0, 1], [1, 1], [2, 1], [0, 2], [1, 2], [2, 2]],
            [[0, 0], [1, 0]],
            [[0, 0], [-1, 0], [-2, 0], [0, 1], [0, 2]],
            [[0, 0], [0, 1], [0, 2], [1, 2], [2, 2]],
        ]


main = Main()


```