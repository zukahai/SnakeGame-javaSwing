## <p align="center"> Làm Game Rắn Săn Mồi Bằng Java </p>
<p align="center"> <img src="https://github.com/zukahai/HaiZuka/blob/master/Images/SnakeGame/1.png" alt="Tieude" /> </p>

### Làm Trò Chơi Rắn Săn Mồi Bằng Java
Chắn hẳn trong chúng ta cũng đã từng chơi trò rắn săn mồi trên chiếc điện thoại đen trắng. Cách chơi của game này rất dễ dàng chỉ cần điều khiển chú rắn làm sao để cho đầu của nó không đâm được vào thân và còn phải ăn những quả bóng điểm để chú rắn lớn hơn.

<p align="center"> <img src="https://github.com/zukahai/HaiZuka/blob/master/Images/SnakeGame/2.png" alt="Snake" /> </p>

### Dữ liệu cần thiết

#### 1. Phân biệt thân rắn, đầu rắn, thức ăn, khoảng trống

Để phần biệt thân rắn, đầu rắn, thức ăn, khoảng trống ta sử dụng một ma trận a.
```Java
	private int a[][] = new int[maxXY][maxXY];
```
Trong đó a[i][j] biểu diễn ô vuông (i, j) thông qua giá trị của nó:

- a[i][j] = 0: Là khoảng trống.
- a[i][j] = 1: Là thân rắn.
- a[i][j] = 2: Là đầu rắn.
- a[i][j] = 3: Là thức ăn.

####2. Thuộc tính của rắn trong trò chơi

Để có thể biểu diễn được con rắn trong game ta cần lưu đồ dài của con rắn (dùng biến sizeSnake) và các tọa độ các ô của thân rắn.
```Java
	private int xSnake[] = new int[maxXY * maxXY];
	private int ySnake[] = new int[maxXY * maxXY];
 ```
 Mỗi cặp tọa độ (xSnake[i], ySnake[i]) là một tọa đọ của thân răn, đặc biệt (xSnake[sizeSnake - 1], ySnake[sizeSnake - 1]) chính là tọa độ của đầu rắn.

Bên cạnh những thuộc tính trên ta cần lưu hướng di chuyển của con rắn bằng các sử dụng biến direction:

- direction =  1 có nghĩa là con rắn đang di chuyển lên trên của màn hình.
- direction =  2 có nghĩa là con rắn đang di chuyển sang phải của màn hình.
- direction =  3 có nghĩa là con rắn đang di chuyển xuống dưới của màn hình.
- direction =  4 có nghĩa là con rắn đang di chuyển sang trái của màn hình.
 
### Thiết lập giao diện

#### 1. Thiết lập giao diện

Phần giao diện của bài này ta sử dụng các button trong class JFrame của Pakage javax.swing

Ta sẽ tạo một ma trận JButton (kích thước m * n) để tạo giao diện cho trò chơi này.
```Java
  	public Container init(int k) {
		Container cn = this.getContentPane();
		pn = new JPanel();
		pn.setLayout(new GridLayout(m,n));
		for (int i = 0; i < m; i++)
			for (int j = 0; j < n; j++){
				bt[i][j] = new JButton();
				pn.add(bt[i][j]);
				bt[i][j].setActionCommand(i + " " + j);
				bt[i][j].addActionListener(this);
				bt[i][j].addKeyListener(this);
				bt[i][j].setBorder(null);
				a[i][j] = 0;
			}
		
		pn2 = new JPanel();
		pn2.setLayout(new FlowLayout());
		
		newGame_bt = new JButton("New Game");
		newGame_bt.addActionListener(this);
		newGame_bt.addKeyListener(this);
		newGame_bt.setFont(new Font("UTM Micra", 1, 15));
		newGame_bt.setBackground(Color.white);
		
		score_bt = new JButton("3");
		score_bt.addActionListener(this);
		score_bt.addKeyListener(this);
		score_bt.setFont(new Font("UTM Micra", 1, 15));
		score_bt.setBackground(Color.white);
		
		for (int i = 1; i <= speed.length; i++)
			lv.addItem("Mức độ " + i);
		lv.setSelectedIndex(k);
		lv.addKeyListener(this);
		lv.setFont(new Font("UTM Micra", 1, 15));
		lv.setBackground(Color.white);
		
		pn2.add(newGame_bt);
		pn2.add(lv);
		pn2.add(score_bt);
		
		a[m / 2][n / 2 - 1] = 1;
		a[m / 2][n / 2] = 1;
		a[m / 2][n / 2 + 1] = 2;
		xSnake[0] = m / 2;
		ySnake[0] = n / 2 - 1;
		xSnake[1] = m / 2;
		ySnake[1] = n / 2;
		xSnake[2] = m / 2;
		ySnake[2] = n / 2 + 1;
		sizeSnake = 3;
		
		creatFood();
		updateColor();
		cn.add(pn);
		cn.add(pn2, "South");
		this.setVisible(true);
		this.setSize(n * 30, m * 30);
		setDefaultCloseOperation(EXIT_ON_CLOSE);
		this.setLocationRelativeTo(null);
		return cn;
	}
```

#### 2. Cập nhật giao diện

Với mỗi bước di chuyển của con rắn ta chỉ cần thay đổi ở ma trận a, và cập nhật lại tất cả màu nền của ma trận button bằng hàm updateColor()
```Java
	public void updateColor() {
		for (int i = 0; i < m; i++)
			for (int j = 0; j < n; j++)
				bt[i][j].setBackground(background_cl[a[i][j]]);
	}
```
Với dãy background_cl[] có giá trị là:
```Java
	Color background_cl[] = {Color.gray, Color.LIGHT_GRAY, Color.darkGray, Color.green};
```
Các màu nền lần lượt là các màu nền của ô trống, thân rắn, đầu rắn, thức ăn của rắn.

### Các hàm xử lý

#### 1. Di chuyển con rắn

Ta thấy rằng khi con rắn di chuyển thì đầu rắn (xác định bới cặp tạo độ (x, y))sẽ di chuyển sang một tọa độ mới, và đuôi rắn sẽ bị đứt (điểm cuối cùng của con rắn sẽ mất đi)

Khi con rắn:

- Đi lên trên thì tọa độ mới của đầu rắn là (x - 1, y).
- Đi sang phải thì tọa độ mới của đầu rắn là (x, y + 1).
- Đi xuống dưới thì tọa độ mới của đầu rắn là (x + 1, y).
- Đi sang trái thì tọa độ mới của đầu rắn là (x, y - 1).
Ta sẽ khởi tạo 2 dãy như sau:
```Java
	int convertX[] = {-1, 0, 1, 0};
	int convertY[] = {0, 1, 0, -1};
```
Nếu (x, y) đang là tọa độ của rắn, và rắn di chuyển theo hướng direction thì tạo độ mới của rắn là (x + covertX[direction  - 1], y + covertY[direction  - 1])

Trong trò chơi con rắn sẽ di chuyển từng nhịp theo một khoảng thời gian (có thể tùy chỉnh).

Ta sử dụng class Timer cho vấn đề này.
```Java
	public snakeGame(String s, int k) {
		super(s);
		cn = init(k);
		timer = new Timer(speed[k], new ActionListener() {
			public void actionPerformed(ActionEvent e) {
				runSnake(direction);
			}
		});
	}
```
Cụ thể hàm runSnake(direction) đầy đủ sẽ như sau:
```Java
	public void runSnake(int k) {
		a[xSnake[sizeSnake - 1]][ySnake[sizeSnake - 1]] = 1;
		xSnake[sizeSnake] = xSnake[sizeSnake - 1] + convertX[k - 1];
		ySnake[sizeSnake] = ySnake[sizeSnake - 1] + convertY[k - 1];
		
		if (xSnake[sizeSnake] < 0)
			xSnake[sizeSnake] = m - 1;
		if (xSnake[sizeSnake] == m)
			xSnake[sizeSnake] = 0;
		if (ySnake[sizeSnake] < 0)
			ySnake[sizeSnake] = n - 1;
		if (ySnake[sizeSnake] == n)
			ySnake[sizeSnake] = 0;
		
		if (a[xSnake[sizeSnake]][ySnake[sizeSnake]] == 1) {
			timer.stop();
			JOptionPane.showMessageDialog(null, "Bạn đã bị thua");
			loss = true;
			return;
		}
		a[xSnake[start]][ySnake[start]] = 0;
		if (xFood == xSnake[sizeSnake] && yFood == ySnake[sizeSnake]) {
			a[xSnake[start]][ySnake[start]] = 1;
			start--;
			creatFood();
			score_bt.setText(String.valueOf(Integer.parseInt(score_bt.getText()) + 1));
		}
		a[xSnake[sizeSnake]][ySnake[sizeSnake]] = 2;
		start++;
		sizeSnake++;
		updateColor();
		for (int i = start; i < sizeSnake; i++) {
			xSnake[i - start] = xSnake[i];
			ySnake[i - start] = ySnake[i];
		}
		sizeSnake -= start;
		start = 0;
	}
```
Trong hàm trên ta thấy có đoạn code là:
```Java
		if (xSnake[sizeSnake] < 0)
			xSnake[sizeSnake] = m - 1;
		if (xSnake[sizeSnake] == m)
			xSnake[sizeSnake] = 0;
		if (ySnake[sizeSnake] < 0)
			ySnake[sizeSnake] = n - 1;
		if (ySnake[sizeSnake] == n)
			ySnake[sizeSnake] = 0;
```
Đoạn này có mục đích là cho rắn có thể chạy "xuyên không" như hình bên dưới.

#### 2. Rắn ăn thức ăn

Viết hàm creatFood() dùng để tạo ra một thức ăn ngẫu nhiên.
```Java
	public void creatFood() {
		int k = 0;
		for (int i = 0; i < m; i++)
			for (int j = 0; j < n; j++)
				if (a[i][j] == 0)
					k++;
		int h = (int) ((k - 1) *  Math.random() + 1);
		k = 0;
		for (int i = 0; i < m; i++)
			for (int j = 0; j < n; j++)
				if (a[i][j] == 0) {
					k++;
					if (k == h) {
						xFood = i;
						yFood = j;
						a[i][j] = 3;
						return;
					}
				}
	}
```
Khi rắn ăn trúng con mồi ta sẽ tạo một thức ăn khác.

Chú ý: khi rắn ăn trúng thức ăn thì đuôi của rắn sẽ không bị đứt.
```Java
		a[xSnake[start]][ySnake[start]] = 0; // cắt đuôi rắn khi rắn di chuyển
		if (xFood == xSnake[sizeSnake] && yFood == ySnake[sizeSnake]) {
			a[xSnake[start]][ySnake[start]] = 1; // nếu rắn ăn trúng thức ăn, nối lại đuổi rắn
			start--;
			creatFood();
			score_bt.setText(String.valueOf(Integer.parseInt(score_bt.getText()) + 1));
		}
```

#### 3. Rắn cắn trúng thân rắn.

Trong lúc di chuyển, nếu đầu rắn trùng tọa độ với thân rắn thì có nghĩa là rắn đã căn trúng thân và bạn đã bị thua, trò chơi kết thúc.
```Java
		if (a[xSnake[sizeSnake]][ySnake[sizeSnake]] == 1) {
			timer.stop();
			JOptionPane.showMessageDialog(null, "Bạn đã bị thua");
			loss = true;
			return;
		}
```
Kết
Trên đây là cách mình đã tạo ra trò chơi rắn săn mồi quen thuộc, do là bài làm cá nhân nên chưa thể tránh hết mọi sai sót, rất mong nhận được góp ý của các bạn ở dưới phần bình luận.

Các bạn có thể tham khảo source code của mình [Tại đây](https://github.com/zukahai/SnakeGame-javaSwing).

Video demo:

[<p align="center"> <img src="https://github.com/zukahai/HaiZuka/blob/master/Images/SnakeGame/3.png" alt="Snake" /> </p>](https://codelearn.io/Media/Default/Users/HaiZuka/HaiZuka/My%20Video.mp4)

## <p align="center">  :tv: Thanks for whatching :earth_africa: </p>
