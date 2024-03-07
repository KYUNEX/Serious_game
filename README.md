using System;
using System.Drawing;
using System.Drawing.Imaging;
using System.Windows.Forms;

namespace JPWP_projekt1
{
    public partial class Form1 : Form
    {
        private Image playerImage;
        private Image plasticBinImage, glassBinImage, paperBinImage;
        private Image[] wasteImages;
        private Image obstacleImage;
        private Image exitIconImage, resetIconImage;
        private Rectangle exitIconArea, resetIconArea;


        private Point playerLocation;
        private Point[] wasteLocations;
        private Point[] obstacleLocations;

        private const int PlayerSize = 80;  
        private const int BinSize = 60;     
        private const int WasteSize = 40;   
        private const int ObstacleSize = 200;

        private int carryingWasteIndex = -1; 
        private int score = 0;

        
        private Rectangle plasticBinArea = new Rectangle(10, 600, BinSize, BinSize);
        private Rectangle glassBinArea = new Rectangle(10, 350, BinSize, BinSize);
        private Rectangle paperBinArea = new Rectangle(10, 100, BinSize, BinSize);

        private bool gameStarted = false;
        private DateTime startTime;
        private TimeSpan gameTime;
        private System.Windows.Forms.Timer gameTimer;
        private bool[] enteredObstacles;
        public Form1()
        {
            InitializeComponent();
            InitializeGame();
            this.KeyPreview = true;
            this.KeyDown += new KeyEventHandler(OnKeyDown);
            this.Size = new Size(1024, 768); // Rozmiar okna

            gameTimer = new System.Windows.Forms.Timer();
            gameTimer.Interval = 1000; // Aktualizacja co sekundę
            gameTimer.Tick += new EventHandler(GameTimer_Tick);
            gameTimer.Start();
           



        }
        private void GameTimer_Tick(object sender, EventArgs e)
        {
            if (gameStarted)
            {
               
                gameTime = DateTime.Now - startTime;

               
                bool allWasteCollected = true;
                foreach (Point wasteLocation in wasteLocations)
                {
                    if (wasteLocation.X >= 0 && wasteLocation.Y >= 0)
                    {
                        allWasteCollected = false;
                        break;
                    }
                }

                
                if (allWasteCollected)
                {
                    gameStarted = false;
                    gameTimer.Stop();
                }

                this.Invalidate(); 
            }
        }



        private void InitializeGame()
        {
            
            playerImage = ResizeImage(Image.FromFile("postac.png"), PlayerSize, PlayerSize);
            plasticBinImage = ResizeImage(Image.FromFile("smietnikplastik.png"), BinSize, BinSize);
            glassBinImage = ResizeImage(Image.FromFile("smietnikszklo.png"), BinSize, BinSize);
            paperBinImage = ResizeImage(Image.FromFile("smietnikpapier.png"), BinSize, BinSize);
            obstacleImage = ResizeImage(Image.FromFile("przeszkoda.png"), ObstacleSize, ObstacleSize);

            exitIconImage = ResizeImage(Image.FromFile("exit.png"), 40, 40);
            resetIconImage = ResizeImage(Image.FromFile("reset.png"), 40, 40);

            exitIconArea = new Rectangle(this.ClientSize.Width - 50, 10, 40, 40);
            resetIconArea = new Rectangle(this.ClientSize.Width - 100, 5, 40, 40);

           
            wasteImages = new Image[12];
            for (int i = 0; i < 4; i++)
            {
                wasteImages[i * 3] = ResizeImage(Image.FromFile("plastik.png"), WasteSize, WasteSize);
                wasteImages[i * 3 + 1] = ResizeImage(Image.FromFile("szklo.png"), WasteSize, WasteSize);
                wasteImages[i * 3 + 2] = ResizeImage(Image.FromFile("papier.png"), WasteSize, WasteSize);
            }

           
            playerLocation = new Point(512, 384);
            wasteLocations = new Point[12];
            Random random = new Random();
            for (int i = 0; i < wasteLocations.Length; i++)
            {
                int x = random.Next(50, this.ClientSize.Width - 50);
                int y = random.Next(50, this.ClientSize.Height - 50);
                wasteLocations[i] = new Point(x, y);
            }

            
            obstacleLocations = new Point[]
            {
        new Point(700, 150),
        new Point(450, 500),
                
            };

            
            enteredObstacles = new bool[obstacleLocations.Length];
            for (int i = 0; i < enteredObstacles.Length; i++)
            {
                enteredObstacles[i] = false;
            }

            this.DoubleBuffered = true; 
        }


        private Image ResizeImage(Image image, int width, int height)
        {
            var destRect = new Rectangle(0, 0, width, height);
            var destImage = new Bitmap(width, height);

            destImage.SetResolution(image.HorizontalResolution, image.VerticalResolution);

            using (var graphics = Graphics.FromImage(destImage))
            {
                graphics.CompositingMode = System.Drawing.Drawing2D.CompositingMode.SourceCopy;
                graphics.CompositingQuality = System.Drawing.Drawing2D.CompositingQuality.HighQuality;
                graphics.InterpolationMode = System.Drawing.Drawing2D.InterpolationMode.HighQualityBicubic;
                graphics.SmoothingMode = System.Drawing.Drawing2D.SmoothingMode.HighQuality;
                graphics.PixelOffsetMode = System.Drawing.Drawing2D.PixelOffsetMode.HighQuality;

                using (var wrapMode = new ImageAttributes())
                {
                    wrapMode.SetWrapMode(System.Drawing.Drawing2D.WrapMode.TileFlipXY);
                    graphics.DrawImage(image, destRect, 0, 0, image.Width, image.Height, GraphicsUnit.Pixel, wrapMode);
                }
            }

            return destImage;
        }

        protected override void OnPaint(PaintEventArgs e)
        {
            base.OnPaint(e);

           
            e.Graphics.DrawImage(playerImage, playerLocation);

           
            if (carryingWasteIndex != -1)
            {
                Point wasteDrawPoint = new Point(playerLocation.X + PlayerSize, playerLocation.Y);
                e.Graphics.DrawImage(wasteImages[carryingWasteIndex], wasteDrawPoint);
            }

            
            e.Graphics.DrawImage(plasticBinImage, plasticBinArea.Location);
            e.Graphics.DrawImage(glassBinImage, glassBinArea.Location);
            e.Graphics.DrawImage(paperBinImage, paperBinArea.Location);
            e.Graphics.DrawImage(exitIconImage, exitIconArea.Location);
            e.Graphics.DrawImage(resetIconImage, resetIconArea.Location);

            
            for (int i = 0; i < wasteImages.Length; i++)
            {
                if (i != carryingWasteIndex) 
                {
                    e.Graphics.DrawImage(wasteImages[i], wasteLocations[i]);
                }
            }

          
            foreach (var loc in obstacleLocations)
            {
                e.Graphics.DrawImage(obstacleImage, loc);
            }

           
            using (Font font = new Font("Arial", 16, FontStyle.Bold))
            using (Brush brush = new SolidBrush(Color.Black))
            using (Brush backgroundBrush = new SolidBrush(Color.Green))
            {
                string scoreText = $"Score: {score}";
                string timeText = $"Time: {gameTime.ToString(@"mm\:ss")}";

                SizeF scoreTextSize = e.Graphics.MeasureString(scoreText, font);
                SizeF timeTextSize = e.Graphics.MeasureString(timeText, font);

                
                float totalWidth = scoreTextSize.Width + timeTextSize.Width + 30;
                float maxHeight = Math.Max(scoreTextSize.Height, timeTextSize.Height) + 10;

                
                RectangleF background = new RectangleF(10, 10, totalWidth, maxHeight);
                e.Graphics.FillRectangle(backgroundBrush, background);

                
                e.Graphics.DrawString(scoreText, font, brush, new PointF(15, 15));

               
                e.Graphics.DrawString(timeText, font, brush, new PointF(15 + scoreTextSize.Width + 10, 15));
            }
        }




        protected override void OnMouseDown(MouseEventArgs e)
        {
            base.OnMouseDown(e);

            Point mouseClickLocation = e.Location;

          
            if (exitIconArea.Contains(mouseClickLocation))
            {
                Application.Exit(); 
            }

           
            if (resetIconArea.Contains(mouseClickLocation))
            {
                ResetGame(); 
            }
        }
        private void ResetGame()
        {
            InitializeGame();
            score = 0; 
            gameTime = TimeSpan.Zero; 
            gameStarted = false; 
            gameTimer.Start(); 
            this.Invalidate(); 
        }



        private void OnKeyDown(object sender, KeyEventArgs e)
        {
            
            if (!gameStarted)
            {
                startTime = DateTime.Now;
                gameStarted = true;
                
            }


            int moveSpeed = 10;
            if (e.KeyCode == Keys.Left || e.KeyCode == Keys.Right || e.KeyCode == Keys.Up || e.KeyCode == Keys.Down)
            {
                CheckCollisionWithObstacles();
            }

            switch (e.KeyCode)
            {
                case Keys.Left:
                    playerLocation.X = Math.Max(0, playerLocation.X - moveSpeed);
                    break;
                case Keys.Right:
                    playerLocation.X = Math.Min(this.ClientSize.Width - playerImage.Width, playerLocation.X + moveSpeed);
                    break;
                case Keys.Up:
                    playerLocation.Y = Math.Max(0, playerLocation.Y - moveSpeed);
                    break;
                case Keys.Down:
                    playerLocation.Y = Math.Min(this.ClientSize.Height - playerImage.Height, playerLocation.Y + moveSpeed);
                    break;
                case Keys.E:
                    HandlePickupOrDrop();
                    break;
            }

            this.Invalidate();
        }
        private void CheckCollisionWithObstacles()
        {
            if (obstacleLocations == null || enteredObstacles == null)
                return;
            Rectangle playerRect = new Rectangle(playerLocation, new Size(PlayerSize, PlayerSize));

            for (int i = 0; i < obstacleLocations.Length; i++)
            {
                int inset = 60;
                Rectangle obstacleRect = new Rectangle(
                    obstacleLocations[i].X + inset,
                    obstacleLocations[i].Y + inset,
                    ObstacleSize - 2 * inset,
                    ObstacleSize - 2 * inset);

                if (playerRect.IntersectsWith(obstacleRect))
                {
                    if (!enteredObstacles[i]) //
                    {
                        score--; 
                        enteredObstacles[i] = true; 
                    }
                }
                else
                {
                    enteredObstacles[i] = false; 
                }
            }

            this.Invalidate(); 
        }


        private void HandlePickupOrDrop()
        {
            if (carryingWasteIndex == -1)
            {
               
                for (int i = 0; i < wasteLocations.Length; i++)
                {
                    if (IsPlayerNearWaste(i))
                    {
                        carryingWasteIndex = i;
                        return;
                    }
                }
            }
            else
            {
                
                if (IsPlayerNearAnyBin())
                {
                    
                    UpdateScore(carryingWasteIndex);
                  
                    wasteLocations[carryingWasteIndex] = new Point(-100, -100);
                    carryingWasteIndex = -1;
                }
            }
        }

        private bool IsPlayerNearWaste(int wasteIndex)
        {
            Rectangle playerRect = new Rectangle(playerLocation, new Size(PlayerSize, PlayerSize));
            Rectangle wasteRect = new Rectangle(wasteLocations[wasteIndex], new Size(WasteSize, WasteSize));
            return playerRect.IntersectsWith(wasteRect);
        }

        private void Form1_Load(object sender, EventArgs e)
        {

        }
        private bool IsPlayerNearAnyBin()
        {
            Rectangle playerRect = new Rectangle(playerLocation, new Size(PlayerSize, PlayerSize));
            return playerRect.IntersectsWith(plasticBinArea) ||
                   playerRect.IntersectsWith(glassBinArea) ||
                   playerRect.IntersectsWith(paperBinArea);
        }
        private bool IsPlayerNearBin()
        {
            Rectangle playerRect = new Rectangle(playerLocation, new Size(PlayerSize, PlayerSize));
            switch (carryingWasteIndex % 3)
            {
                case 0: return playerRect.IntersectsWith(plasticBinArea);
                case 1: return playerRect.IntersectsWith(glassBinArea);
                case 2: return playerRect.IntersectsWith(paperBinArea);
                default: return false;
            }
        }

        private void UpdateScore(int wasteIndex)
        {
            Rectangle playerRect = new Rectangle(playerLocation, new Size(PlayerSize, PlayerSize));
            bool correctBin = false;

            switch (wasteIndex % 3)
            {
                case 0: 
                    correctBin = playerRect.IntersectsWith(plasticBinArea);
                    break;
                case 1: 
                    correctBin = playerRect.IntersectsWith(glassBinArea);
                    break;
                case 2: 
                    correctBin = playerRect.IntersectsWith(paperBinArea);
                    break;
            }

            if (correctBin)
            {
                score++; 
            }
            else
            {
                score--; 
            }
        }
    }
}
