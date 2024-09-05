using System;
using System.Net.Http;
using System.Text.RegularExpressions;
using System.Threading.Tasks;

public class LinkMinimizer
{
    private const string ApiKey = "YOUR_API_KEY"; // Replace with your actual API key
    private const string ApiUrl = "https://api.shrtco.de/v2/shorten";

    public static async Task Main(string[] args)
    {
        Console.WriteLine("Welcome to the Link Minimizer!");
        Console.WriteLine("Enter a URL to shorten:");

        string url = Console.ReadLine();

        if (IsValidUrl(url))
        {
            string shortenedUrl = await ShortenUrl(url);

            if (!string.IsNullOrEmpty(shortenedUrl))
            {
                Console.WriteLine($"Shortened URL: {shortenedUrl}");
            }
            else
            {
                Console.WriteLine("Error shortening URL. Please try again.");
            }
        }
        else
        {
            Console.WriteLine("Invalid URL. Please enter a valid URL.");
        }

        Console.ReadKey();
    }

    private static async Task<string> ShortenUrl(string url)
    {
        try
        {
            using (HttpClient client = new HttpClient())
            {
                var requestData = new
                {
                    original_link = url
                };

                var content = new StringContent(System.Text.Json.JsonSerializer.Serialize(requestData), Encoding.UTF8, "application/json");

                var response = await client.PostAsync(ApiUrl, content);

                if (response.IsSuccessStatusCode)
                {
                    var responseContent = await response.Content.ReadAsStringAsync();
                    var responseObject = System.Text.Json.JsonSerializer.Deserialize<ShortenResponse>(responseContent);
                    return responseObject.result.full_short_link;
                }
                else
                {
                    Console.WriteLine($"Error: {response.StatusCode}");
                    return null;
                }
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error shortening URL: {ex.Message}");
            return null;
        }
    }

    private static bool IsValidUrl(string url)
    {
        // Basic URL validation (replace with a more robust implementation if needed)
        string pattern = @"https?://(www.)?[-a-zA-Z0-9@:%._+~#=]{1,256}.[a-zA-Z0-9()]{1,6}b([-a-zA-Z0-9()@:%_+.~#?&//=]*)";
        return Regex.IsMatch(url, pattern, RegexOptions.IgnoreCase);
    }

    // Response model for the API
    private class ShortenResponse
    {
        public Result result { get; set; }
    }

    private class Result
    {
        public string full_short_link { get; set; }
    }
}
