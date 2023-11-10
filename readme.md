Wifi password


using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Text.RegularExpressions;
using System.Net;

namespace WifiPasswordGetter
{
    class Program
    {
        static void Main(string[] args)
        {
            List<string> wifiNetworks = GetWifiPassword();

            foreach (string network in wifiNetworks)
            {
                Console.WriteLine($"Network: {network}");
                string password = GetWifiPassword(network);
                Console.WriteLine($"Password: {password}");
                Console.WriteLine();
            }
        }

        private static List<string> GetWifiPassword()
        {
            ProcessStartInfo startInfo = new ProcessStartInfo("netsh", "wlan show networks mode=bssid")
            {
                RedirectStandardOutput = true,
                UseShellExecute = false,
                CreateNoWindow = true
            };

            Process process = new Process { StartInfo = startInfo };
            process.Start();

            string output = process.StandardOutput.ReadToEnd();
            process.WaitForExit();

            string[] lines = output.Split(new string[] { "\n" }, StringSplitOptions.RemoveEmptyEntries);

            List<string> wifiNetworks = new List<string>();

            foreach (string line in lines)
            {
                if (line.Contains("SSID"))
                {
                    string ssid = Regex.Match(line, "SSID\\s+:\\s+(.+)").Groups[1].Value;
                    wifiNetworks.Add(ssid);
                }
            }

            return wifiNetworks;
        }

        private static string GetWifiPassword(string profile)
        {
            ProcessStartInfo startInfo = new ProcessStartInfo("netsh", $"wlan show profile name=\"{profile}\" key=clear")
            {
                RedirectStandardOutput = true,
                UseShellExecute = false,
                CreateNoWindow = true,
            };

            Process process = new Process { StartInfo = startInfo };
            process.Start();

            string output = process.StandardOutput.ReadToEnd();
            process.WaitForExit();

            string[] lines = output.Split(new string[] { "\n" }, StringSplitOptions.RemoveEmptyEntries);

            foreach (string line in lines)
            {
                if (line.Contains("Key content"))
                {
                    return line.Substring(line.IndexOf(':') + 1).Trim();
                }
            }
              return "No password found";
        }
    }
}
