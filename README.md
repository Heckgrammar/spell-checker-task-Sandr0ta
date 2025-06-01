string[] dictionary = createDictionary();
            HashSet<string> wordSet = new HashSet<string>(dictionary, StringComparer.OrdinalIgnoreCase);

            Console.WriteLine("Enter a word or sentence:");
            string userText = Console.ReadLine(); 

            string[] inputWords = userText.Split(' ', StringSplitOptions.RemoveEmptyEntries);

           
            List<string> correctWords = new List<string>();
            List<string> incorrectWords = new List<string>();

            foreach (string word in inputWords)
            {
                string normalizedWord = word.Trim().ToLower(); 
                if (wordSet.Contains(normalizedWord))
                    correctWords.Add(normalizedWord);
                else
                    incorrectWords.Add(normalizedWord);
            }

            
            Console.WriteLine("\nCorrectly spelled words:");
            Console.WriteLine(string.Join(", ", correctWords));

            Console.WriteLine("\nIncorrectly spelled words:");
            Console.WriteLine(string.Join(", ", incorrectWords));

            double score = (double)correctWords.Count / inputWords.Length * 100;
            Console.WriteLine($"\nSpelling Score: {score:0.00}%");

            
            File.WriteAllLines("IncorrectWords.txt", incorrectWords);
            Console.WriteLine("\nIncorrect words saved to IncorrectWords.txt");

            
            List<string> suggestions = new List<string>();
            foreach (string wrongWord in incorrectWords)
            {
                string suggestion = GetClosestMatch(wrongWord, wordSet);
                if (suggestion != null)
                {
                    Console.WriteLine($"Did you mean '{suggestion}' instead of '{wrongWord}'?");
                    suggestions.Add(suggestion);
                }
            }

            if (suggestions.Any())
            {
                File.WriteAllLines("SuggestionsList.txt", suggestions);
                Console.WriteLine("\nSuggestions saved to SuggestionsList.txt");
            }
        }

        static string[] createDictionary()
        {
            using StreamReader reader = new("WordsFile.txt");
            List<string> words = new List<string>();
            while (!reader.EndOfStream)
            {
                words.Add(reader.ReadLine());
            }
            return words.ToArray();
        }

        
        static string GetClosestMatch(string inputWord, HashSet<string> dictionary)
        {
            int minDistance = int.MaxValue;
            string closestWord = null;

            foreach (string word in dictionary)
            {
                int distance = LevenshteinDistance(inputWord, word);
                if (distance < minDistance && distance <= 2) // adjustable threshold
                {
                    minDistance = distance;
                    closestWord = word;
                }
            }

            return closestWord;
        }

        static int LevenshteinDistance(string a, string b)
        {
            int[,] dp = new int[a.Length + 1, b.Length + 1];

            for (int i = 0; i <= a.Length; i++) dp[i, 0] = i;
            for (int j = 0; j <= b.Length; j++) dp[0, j] = j;

            for (int i = 1; i <= a.Length; i++)
            {
                for (int j = 1; j <= b.Length; j++)
                {
                    int cost = a[i - 1] == b[j - 1] ? 0 : 1;
                    dp[i, j] = Math.Min(Math.Min(
                        dp[i - 1, j] + 1,
                        dp[i, j - 1] + 1),
                        dp[i - 1, j - 1] + cost);
                }
            }

            return dp[a.Length, b.Length];
