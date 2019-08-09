using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class Program
    {
        public static Random rng = new Random();
        static void Main(string[] args)
        {
            PopeGame game = new PopeGame();

            Console.WriteLine("Press 1 for Pope Bruce\nPress 2 for Bayes demonstration");
            char MenuChoice;
            while (true)
            {
                MenuChoice = Console.ReadKey(true).KeyChar;
                if (MenuChoice == '1') { game.StartGame(); return; }
                if (MenuChoice == '2') { break; }
            }
            Console.Clear();
            GameData notAFuckingGame = new GameData(0, 0, 0, 0);
            int NumberOfTest = 200000;
            decimal TestFalseReading = 100; // 1 in 100 wrong, 99% reliable
            decimal ProbabilityOfBlack = 200; //1 in 200 are black;
            notAFuckingGame.TurncMode = false;
            notAFuckingGame.NeverJudgeSinnerInnocent = true;
            notAFuckingGame.OneInXFault = TestFalseReading;
            notAFuckingGame.OneInXSinful = ProbabilityOfBlack;
            notAFuckingGame.ReliabilityLimit = false;

            int FalsePositive = 0;
            int TruePositive = 0;
            for (int i = 0; i < NumberOfTest; i++)
            {
                //Console.Write(".");
                int RandomNumber = rng.Next(0, (int)ProbabilityOfBlack);
                bool IsBlack = (RandomNumber == 0);
                int TestResult = rng.Next(0, (int)TestFalseReading);
                bool TestWrong = (TestResult == 0);

                if (TestWrong && !IsBlack)
                {
                    FalsePositive++;
                }
                if (!TestWrong && IsBlack)
                {
                    TruePositive++;
                }
            }

            Console.WriteLine("The Test will be wrong 1 in " + TestFalseReading + " times. It is " + (100 - (decimal)100.0f / TestFalseReading) + "% accurate!");
            Console.WriteLine("The Odds of an idividual being Positive is 1 in " + ProbabilityOfBlack + ", aka " + (decimal)100.0 / ProbabilityOfBlack + "%!");
            Console.WriteLine("\n##### Pointless data #####");
            Console.WriteLine("Number of Tests: " + NumberOfTest);
            Console.WriteLine("Number of Positives: " + (TruePositive + FalsePositive));
            Console.WriteLine("Number of False Positives: " + FalsePositive);
            Console.WriteLine("Number of True Positives: " + TruePositive);

            Console.WriteLine("\nThe Question Is: \"If Given A Positive Result, What Percent Chance Is There Of The Outcome Being A TRUE POSITIVE?\"");

            Console.ReadKey(true);

            Console.WriteLine("The ANSWER IS...");
            decimal TheAnswer = (decimal)TruePositive / ((decimal)FalsePositive + (decimal)TruePositive);
            Console.WriteLine(TheAnswer * 100 + "% !!!!!");
            Console.ReadKey(true);
            decimal localpBA = (1 - 1 / TestFalseReading);
            decimal localpA = (1 / ProbabilityOfBlack);
            decimal localpB = localpBA*localpA + (1 - 1 / ProbabilityOfBlack) * (1 / TestFalseReading);
            decimal localBayes = (localpBA * localpA) / localpB;
            Console.WriteLine("\n If this was the Pope game, given the rate of true positives in a population" +
                "\n and the rate of false positives, the chance that any particualr mortal would be a sinner" +
                "\n would be {0}%!" +
                "\n But assuming false negatives were as likely as false positives, as in this demo," +
                "\n the value ought to be {1}" +
                "\n\n That value should be rather close to the answer here!" +
                "\n The value was computed via Bayes' formula (P(A|B)=(P(B|A)*P(A))/P(B))..." +
                "\n P(A|B) is 'given B is true, how likely is A true?" +
                "\n P(A) is 'Is Sinner', P(B) is 'Judged Sinner'. P(B|A) is the chance a sinner is judged a sinner" +
                " \n\nPress any key to exit program...", notAFuckingGame.Bayes * 100, localBayes*100);
            Console.ReadKey(true);
        }


    }

    public class PopeGame
    {
        Random rng;
        public PopeGame()
        {
            rng = new Random();
        }

        public void StartGame()
        {
            Console.Clear();
            MainMenu();
            bool GameActive = true;
            GameData popeData = new GameData(5, -10, -10, 5);
            while (GameActive)
            {
                Console.Clear();
                GameDisplay(popeData);

                decimal OneInXSinful = TownSinfulness();
                decimal OneInXFault = TestReliability();
                popeData.OneInXFault = OneInXFault;
                popeData.OneInXSinful = OneInXSinful;
                //decimal PercSinful = 1.0f / OneInXSinful * 100.0f;
                //decimal PercReliable = 100.0f - (1.0f / OneInXFault * 50.0f);
                //decimal Bayes = (PercReliable * PercSinful) / (PercReliable * PercSinful + (100-PercReliable) * (100 - PercSinful)) ;
                bool Sinner = (10000 >= rng.Next(0, (int)(10000 / popeData.Bayes)));
                popeData.Sinner = Sinner;

                DisplayJudgementQuestion(popeData);

                bool JudgedSinner;
                while (true)
                {
                    char input = Console.ReadKey(true).KeyChar;
                    if (input == '1')
                    {
                        JudgedSinner = false;
                        break;
                    }
                    if (input == '2')
                    {
                        JudgedSinner = true;
                        break;
                    }
                    if (input == 'p' || input == 'P')
                    {
                        popeData.PopeVision = !popeData.PopeVision;
                        Console.Clear();
                        while (Console.KeyAvailable) { Console.ReadKey(true); }
                        GameDisplay(popeData);
                        DisplayJudgementQuestion(popeData);
                    }
                }
                Console.Clear();
                while (Console.KeyAvailable) { Console.ReadKey(true); }
                GameDisplay(popeData);
                decimal ScoreStart = popeData.Score;
                if (Sinner == JudgedSinner)
                {
                    if (Sinner) { Console.WriteLine("A sinner hath been sent to hell!"); popeData.Score += popeData.SinnerHell; }
                    else { Console.WriteLine("You sent a saint to Heaven! Amen!"); popeData.Score += popeData.SaintHeaven; }
                }
                if (Sinner != JudgedSinner)
                {
                    if (Sinner) { Console.WriteLine("Ye dolt! You sent a sinner to paradise!"); popeData.Score += GodForgives(popeData.Bayes, popeData.SinnerHeaven); }
                    else { Console.WriteLine("You sent a saint to hell, damned fool!"); popeData.Score += GodForgives((decimal)1.0 - popeData.Bayes, popeData.SaintHell); }
                }
                Console.WriteLine("You {0} {1} points!", (ScoreStart > popeData.Score) ? "lost" : "gained", Math.Abs(ScoreStart-popeData.Score));
                Console.WriteLine("\n (There was a " + popeData.Bayes * 100 + "% chance that they were a sinner!");
                Console.WriteLine("\n Press any key to continue...");
                Console.ReadKey(true);
                popeData.Judgements++;
                if (popeData.Score >= 100 || popeData.Score <= 0)
                {
                    GameActive = false;
                }
            }
            Console.Clear();
            if (popeData.Score >= 100)
            {
                Console.WriteLine("You have done well! Congrats! You get an extra large hat, straight from God himself!");
                if(popeData.Cheater)
                {
                    Console.WriteLine("Next time, try doing it without using PopeVision!");
                }
                else { Console.WriteLine("And you did it without using PopeVision! Truly, YOU are most highest!"); }
            }
            else
            {
                Console.WriteLine("\"YE WHO HATH FUCKED UP MUCH, BEGONE FROM MY SIGHT!\" \nGod throws you from your pope seat, and straight into the pits of hell! WHOOPS");
            }
            Console.ReadKey(true);
        }

        public void MainMenu()
        {
            Console.WriteLine("                      POPE BRUCE\n           Turning Bayes' Theorem into an Ecumenical Matter!");
            Console.WriteLine("\n Do you have what it takes to Pope, or are you just too pooped to Pope?\n " +
                "It is Judgement Day, and God has sent his legions to hunt out all the Earth's sinners! \n " +
                "Your job as Pope is to decide who gets to kick back in paradise and who burns in the lake of fire.\n\n " +
                "Angels only bring forth those they have judged to be sinners, and while they never judge a guilty man innocent,\n " +
                "they sometimes are a bit overzealous and will bring you saints they've mistaken for sinners!\n\n " +
                "It may be odd for a Pope to be accountable, but God has decided that YOU will make the final judgement " +
                "for these souls, as proof of your Pope qualifications! \n " +
                "(That, and God is absolutely SWAMPED since there's the whole end times going on) \n " +
                "But beware, God is forgiving of bad luck, but He shall punish those that fucketh up simple calls!\n\n " +
                "Press any key to begin...");
            Console.ReadKey(true);
        }

        public void GameDisplay(GameData data)
        {
            Console.WriteLine("+++POPE BRUCE+++");
            Console.WriteLine("POPE RATING: " + data.Score + "   JUDGEMENTS: " + data.Judgements);
            Console.WriteLine("Score for Sending Sanits to Heaven: " + data.SaintHeaven);
            Console.WriteLine("Score for Sending Sanits to Hell: Up to " + data.SaintHell);
            Console.WriteLine("Score for Sending Sinners to Heaven: Up to " + data.SinnerHeaven);
            Console.WriteLine("Score for Sending Sinners to Hell: " + data.SinnerHell);
            Console.WriteLine("- - - - - - - - - - - - -");
        }
        public decimal TownSinfulness()
        {
            decimal OneIn = rng.Next(1, 51);
            return OneIn;
        }

        public decimal TestReliability()
        {
            decimal OneIn = rng.Next(1, 26);
            return OneIn;
        }

        public decimal GodForgives(decimal chanceOfFailure, decimal penelty)
        {
            return (decimal)Math.Round(penelty*(decimal)0.5 + (Math.Truncate(chanceOfFailure*100) * penelty)/200);
        }

        public void DisplayJudgementQuestion(GameData data)
        {
            Console.WriteLine("An Angel brings forth a mortal whom they have deemed a sinner!");
            Console.WriteLine("The mortal is from a place where " + data.PercSinful + "% of its inhabitants are dirty sinners!");
            Console.WriteLine("This Angel is known for being correct in their judgements " + data.PercReliable + "% of the time.");

            Console.WriteLine("\nWHAT BE YE JUDGEMENT, POPE?");
            Console.WriteLine("\n Press 1 to send them to heaven.");
            Console.WriteLine(" Press 2 to send them to hell!");
            Console.WriteLine(" Press P to {0} POPE VISION (cheat mode)", data.PopeVision ? "deactivate":"activate");

            if (data.PopeVision)
            {
                Console.WriteLine("\n POPEVISION DATA\n" +
                    " Chance of Sinner: " + data.Bayes * 100 +
                    "\n {0}", data.Sinner ? "Sinner":"Saint");
            }
        }
    }
}

public class GameData
{

    public decimal Score;
    public decimal SinnerHell;
    public decimal SaintHell;
    public decimal SaintHeaven;
    public decimal SinnerHeaven;
    public decimal Judgements;
    public bool PopeVision { get { return _PopeVision; } set { if (value == true) { Cheater = true; } _PopeVision = value; } }
    private bool _PopeVision;

    public bool NeverJudgeSinnerInnocent;

    public bool Cheater;
    public bool ReliabilityLimit; // if true, realiability has a floor of 50%

    public decimal OneInXSinful;
    public decimal OneInXFault;
    public decimal PercSinful { get { return TurncMode ? _PercSinulTurnc : _PercSinful; } }
    public decimal PercReliable { get { return TurncMode ? _PercReliableTurnc : _PercReliable; } }
    
    decimal _PercSinful { get { return 1 / OneInXSinful * 100; } }

    decimal _PercReliable
    { get
        {
            if (ReliabilityLimit)
            {
                return  100 - (1 / OneInXFault * 50);
            }
            return (100 - 1 / OneInXFault * 100);
        }
    }

    decimal _PercSinulTurnc { get { return TurncABit(_PercSinful); } }
    decimal _PercReliableTurnc { get { return TurncABit(_PercReliable); } }

    decimal TurncABit(decimal number)
    {
        return (decimal)Math.Truncate((double)number * 10) / (decimal)10.0;
    }

    public decimal Bayes { get { return pBApA/PB; } }

    decimal pBA
    { get
        {
            if (NeverJudgeSinnerInnocent) { return 1; }
                return PercReliable/100; }
    } // Chance a sinner is judged a sinner. p(+ | Sin)

    decimal pA { get { return PercSinful/100; } } // Chance of a random mortal being a sinner. p(Sin)
    decimal pBnA { get { return 1 - PercReliable / 100; } } // Chance of false positive. p(+ | Saint)
    decimal pnA { get { return 1 - PercSinful / 100; } } // Chance of random mortal not being a sinner. p(Saint)
    decimal pBApA { get { return pBA * pA; } }  // Chance a sinner is judged a sinner, multiplied by the chance of a sinner
    decimal PB { get { return pBApA + pBnA * pnA; } } //Chance of (+ | S) + Chance of (+ | Saint) ; total prob. of getting a positive 

    public bool Sinner;

    public bool TurncMode;

    public GameData(decimal sinnerToHell, decimal sinnerToHeaven, decimal saintToHell, decimal saintToHeaven)
    {
        Score = 50;
        Judgements = 0;
        SinnerHell = sinnerToHell;
        SaintHell = saintToHell;
        SaintHeaven = saintToHeaven;
        SinnerHeaven = sinnerToHeaven;
        PopeVision = false;
        ReliabilityLimit = true;
        NeverJudgeSinnerInnocent = true;

        TurncMode = true;

        OneInXSinful = 1;
        OneInXFault = 1;
        Sinner = true;
    }
}
