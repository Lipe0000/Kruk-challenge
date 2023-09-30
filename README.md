using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.IO;
using System.Security.Cryptography;
using System.Threading;
using System.Timers;

namespace KruK
{
    class Program
    {
        static string targetDirectory = "C:\\Path\\To\\TargetDirectory";
        static string backupDirectory = "C:\\Path\\To\\BackupDirectory";
        static List<string> knownFiles = new List<string>
        {
            "file1.txt",
            "file2.docx"
        };

        static HashSet<string> whitelistedProcesses = new HashSet<string>
        {
            "trustedprocess1.exe",
            "trustedprocess2.exe"
        };

        // Substitua esta linha com o valor real do hash conhecido (SHA-256)
        static string knownHashValue = "ColoqueAquiOSeuValorDeHashConhecido";

        static void Main(string[] args)
        {
            StartScheduledChecks();

            // Mantém a aplicação em execução indefinidamente
            Console.WriteLine("Pressione Ctrl+C para sair.");
            Thread.Sleep(Timeout.Infinite);
        }

        static void StartScheduledChecks()
        {
            System.Timers.Timer timer = new System.Timers.Timer();
            timer.Elapsed += CheckForRansomware;
            timer.Interval = TimeSpan.FromHours(1).TotalMilliseconds;
            timer.Start();
        }

        static void CheckForRansomware(object sender, ElapsedEventArgs e)
        {
            foreach (var filePath in knownFiles)
            {
                string fullFilePath = Path.Combine(targetDirectory, filePath);
                if (File.Exists(fullFilePath))
                {
                    string calculatedHash = CalculateFileHash(fullFilePath);

                    if (calculatedHash == knownHashValue)
                    {
                        Console.WriteLine($"Fake file {filePath} has been encrypted!");
                        BackupFiles();
                        TakeActionsOnSuspiciousActivity();
                    }
                }
            }

            MonitorProcesses();
        }

        static string CalculateFileHash(string filePath)
        {
            using (var sha256 = SHA256.Create())
            using (var stream = File.OpenRead(filePath))
            {
                byte[] hashBytes = sha256.ComputeHash(stream);
                return BitConverter.ToString(hashBytes).Replace("-", "").ToLower();
            }
        }

        static void BackupFiles()
        {
            foreach (var filePath in knownFiles)
            {
                string sourcePath = Path.Combine(targetDirectory, filePath);
                string destinationPath = Path.Combine(backupDirectory, filePath);

                if (File.Exists(sourcePath))
                {
                    // Copia o arquivo para o diretório de backup (simulação de backup)
                    File.Copy(sourcePath, destinationPath, true);
                    Console.WriteLine($"Arquivo {filePath} foi copiado para o diretório de backup.");
                }
            }
        }

        static void TakeActionsOnSuspiciousActivity()
        {
            Console.WriteLine("Tomando medidas contra atividade suspeita...");

            SuspendMaliciousProcesses();

            // Adicione mais ações conforme necessário
        }

        static void SuspendMaliciousProcesses()
        {
            string[] maliciousProcessNames = { "malicious.exe", "ransom.exe" };

            foreach (string processName in maliciousProcessNames)
            {
                Process[] processes = Process.GetProcessesByName(Path.GetFileNameWithoutExtension(processName));
                foreach (Process process in processes)
                {
                    try
                    {
                        process.Kill();
                        Console.WriteLine($"Processo malicioso '{process.ProcessName}' encerrado com sucesso.");
                    }
                    catch (Exception ex)
                    {
                        Console.WriteLine($"Erro ao encerrar o processo: {ex.Message}");
                    }
                }
            }
        }

        static void MonitorProcesses()
        {
            Process[] allProcesses = Process.GetProcesses();
            foreach (Process process in allProcesses)
            {
                if (!whitelistedProcesses.Contains(process.ProcessName.ToLower()))
                {
                    Console.WriteLine($"Processo suspeito detectado: {process.ProcessName}");
                    TakeActionsOnSuspiciousActivity();
                }
            }
        }
    }
}
