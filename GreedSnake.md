# CyanTown
+ import java.util.ArrayList;
+ import java.util.List;
+ import javax.swing.JFrame;
+ import java.awt.Color;
+ import java.awt.Graphics;
+ import java.awt.Graphics2D;
+ import java.awt.Rectangle;
+ import java.awt.event.KeyAdapter;
+ import java.awt.event.KeyEvent;
+ import java.awt.image.BufferedImage;

public class SnakeGame extends JFrame {
	public static final int WIDTH = 800, HEIGHT = 600, SLEEPTIME = 200,
							L = 1,R = 2, U = 3, D = 4;	//设置窗口大小，方向键代值
	public int grade;
	BufferedImage offersetImage= new BufferedImage( WIDTH, HEIGHT, BufferedImage.TYPE_3BYTE_BGR);
	//创建一个大小为 800*600 ，且具有 8 位 RGB 颜色分量的图像变量
	Rectangle rect = new Rectangle(15, 45, 15 * 50, 15 * 35);
	//创建一个以 (15, 45) 为坐上顶点，大小为750*525的矩形变量
	Snake snake;
	SNode node;
	
	public SnakeGame() {								//修改 SnakeGame 类的默认构造方法
		snake = new Snake(this);
		createNode();									//随机位置创建第一个食物
		this.setBounds(0, 0, WIDTH, HEIGHT);			// JFrame 设置窗口大小
		this.addKeyListener(new KeyAdapter() {			//匿名类添加键盘监听事件
			public void keyPressed(KeyEvent arg0) {						
				switch (arg0.getKeyCode()) {
					case KeyEvent.VK_LEFT:
						if (snake.dir == R) snake.dir = R;				//不许反向
						else snake.dir = L;
						break;
					case KeyEvent.VK_RIGHT:
						if (snake.dir == L) snake.dir = L;
						else snake.dir = R;
						break;
					case KeyEvent.VK_UP:
						if (snake.dir == D) snake.dir = D;
						else snake.dir = U;
						break;
					case KeyEvent.VK_DOWN:
						if (snake.dir == U) snake.dir = U; 
						else snake.dir = D;
						break;
				}	//	Updated in 9-4
			}
		});
		this.setTitle("Snake Game");									//窗口标题
		this.setDefaultCloseOperation(EXIT_ON_CLOSE);					//关闭方式
		this.setVisible(true);											//显示窗口
		ThreadUpadte threadupadte = new ThreadUpadte();
		Thread thread = new Thread(threadupadte);
		thread.start();													//启动线程
		
	}

	public void paint(Graphics g) {
		Graphics2D g2d = (Graphics2D) offersetImage.getGraphics();		//强制类型转换
		g2d.setColor(Color.white);										//设置填充颜色
		g2d.fillRect(0, 0, WIDTH, HEIGHT);								//将背景填充为白色
		g2d.setColor(Color.black);										//设置画矩形颜色
		g2d.drawRect(rect.x, rect.y, rect.width, rect.height);			//画出大矩形
		g2d.drawString(" 得分：" + grade, 15, 585);						//画出分数		
		if (snake.hitFood(node)) {											
			createNode();
			grade ++;
		}																//吃到食物后创建新的食物
		snake.draw(g2d);	//画蛇
		node.draw(g2d);		//画食物
		g.drawImage(offersetImage, 0, 0, this);							//显示所有作图
	}

	class ThreadUpadte implements Runnable {			// ThreadUpadte 实现 Runnable 接口
		public void run() {								
			while (true) {
				try {
					Thread.sleep(SLEEPTIME);
					repaint();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}				// 线程休息方法 sleep 与 重复作图方法 repaint 无限调用实现蛇的前进
	}
	
	public void createNode() {
		int a = (int) (Math.random() * 48) + 1;			//随机产生 1 - 49 的数字	（别问我为什么数字如此
		int	b = (int) (Math.random() * 31) + 3;			//随机产生 3 - 34 的数字	  奇葩，因为有BUG！！）
		int x = 15*a;										 
		int y = 15*b;									//横，纵坐标各放大十五倍以适应坐标
		//因食物为 15*15 的方块，故采用此方法让食物与蛇横纵对齐
		node = new SNode(x, y, Color.blue);
	}													//随机位置创建食物
	
	public static void main(String args[]) {
		new SnakeGame();
	}
}

class SNode {
	int x, y, width = 15, height = 15;							//蛇身以及食物的大小为15*15
	Rectangle rect = new Rectangle(x, y, width, height);		//创建蛇身或者食物的方块对象
	Color color;
	
	public SNode(int x, int y, Color color) {					//构造方法：方块左上角点的坐标及颜色
		this.x = x; this.y = y;									//添加颜色是为了区分蛇身与食物
		this.color = color;
	}
	
	public void draw(Graphics2D g2d) {							//画出蛇身或者食物方块
		g2d.setColor(color);
		g2d.drawRect(x, y, width, height);
	}
	
	public Rectangle getRect() {								//得到目前方块左上角点的坐标
		rect.x = x; rect.y = y;
		return rect;
	}
}

class Snake {   
	public List<SNode> nodes = new ArrayList<SNode>();			//用链表来存储蛇身
	SnakeGame interFace;
	int dir=SnakeGame.R;										//默认方向为右
	
	
	public Snake(SnakeGame interFace) {							
		this.interFace = interFace;
		nodes.add(new SNode(150, 150, Color.black));			//蛇的起始位置
		addNode();												//蛇头添加一个方块使初始长度为2
	}
	
	public boolean hitFood(SNode node) {						//判断蛇是否装上食物
		if (nodes.get(0).getRect().intersects(node.getRect())) {//判断两矩形是否相交
			addNode();											//相交即吃到在蛇头添加一个方块
			return true;						
		}
		return false;
	}
	
	public boolean hitWall(SNode node){		
		if (node.getRect().intersects(new Rectangle(15, 45, 15 * 50, 15 * 35))) 
			return false;
		/* 如果两矩形相交则表示未撞上墙，小矩形跑出大矩形则已撞上墙
		 * intersects() 指两个矩形整体，并非单指外边缘线 */
		else return true;
	}		//   Updated in 9-4
	
	public boolean eatSelf(SNode node){
		for (int i = 2; i < nodes.size(); i++)
			if (node.getRect().intersects(nodes.get(i).getRect())) 
				return true;
		// for 循环判定蛇头方块是否与蛇身相交
		return false;
	}		//	Updated in 9-4
	
	public void draw(Graphics2D g2d) {					//画蛇
		if (!hitWall(nodes.get(0)) && !eatSelf(nodes.get(0))) {
		// 每次画蛇前判断是否撞墙或吃自己，若有一为真则停止
			for (int i = 0; i < nodes.size(); i++)
					nodes.get(i).draw(g2d);				//用 nodes.size() 返回蛇长，画出符合蛇长的方块
			move();										//蛇的移动
		}
		else {
			g2d.setBackground(Color.BLUE);
			g2d.clearRect(15, 45, 15 * 50, 15 * 35);
			for (int i=0; i <= 5; i++) {
				g2d.drawString("Failed", 200 + i*50, 300);
			}
		}
	}
	
	public void move() {							//蛇的移动
		nodes.remove((nodes.size() - 1));			//移除一个蛇尾方块
		addNode();									//添加一个蛇头方块
	}
	
	public synchronized void addNode() {			//添加一个蛇头方块
		SNode tempNode = nodes.get(0);			//将蛇头方块赋给 nodeTempNode 
		switch (dir) {								
		case SnakeGame.L:
			//if (tempNode.x <= 15) 
				//tempNode = new SNode(15 + 15 * 50, tempNode.y, Color.black);
				//return;
			nodes.add(0, new SNode(tempNode.x - tempNode.width, tempNode.y, Color.black ));
			break;
		case SnakeGame.R:
			//if (tempNode.x >= 5 + 15 * 50 - tempNode.width) 
				//tempNode = new SNode(15 - tempNode.width, tempNode.y, Color.black);
			nodes.add(0, new SNode(tempNode.x + tempNode.width, tempNode.y, Color.black));
			break;
		case SnakeGame.U:
			//if (tempNode.y <= 45) 
				//tempNode = new SNode(tempNode.x, 45 + 15 * 35, Color.black);
			nodes.add(0, new SNode(tempNode.x, tempNode.y - tempNode.height, Color.black));
			break;
		case SnakeGame.D:
			//if (tempNode.y >= 45 + 15 * 35 - tempNode.height) 
				//tempNode = new SNode(tempNode.x,45 - tempNode.height, Color.black);
			nodes.add(0, new SNode(tempNode.x, tempNode.y + tempNode.height, Color.black));
			break;
		}			//根据不同方向，若撞墙则从墙的另一端出现，若未撞墙则在蛇头位置添加一个方块
					//Updated in 9-4 : 撞墙不再从另一端出现
	}
}
