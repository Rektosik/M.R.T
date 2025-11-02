```
using System;
using System.Collections.Generic;
using System.Globalization;

class VarItem
{
    public string id;
    public double a;
    public double c;
    public double x1;
    public double x2;

    public VarItem(string id, double a, double c, double x1, double x2)
    {
        this.id = id;
        this.a = a;
        this.c = c;
        this.x1 = x1;
        this.x2 = x2;
    }
}

class Program
{
    static void Main()
    {
        CultureInfo.CurrentCulture = CultureInfo.InvariantCulture;

        var list = new List<VarItem>();
        list.Add(new VarItem("1", 10, 2.0, 0, 10));
        list.Add(new VarItem("2", 20, 3.0, 10, 20));
        list.Add(new VarItem("3", 50, 0.1, 20, 30));
        list.Add(new VarItem("4", 30, 1.0, 5, 10));
        list.Add(new VarItem("5", 45, 0.5, 5, 20));
        list.Add(new VarItem("6", 55, 4.0, 5, 30));
        list.Add(new VarItem("7", 25, 0.25, 0, 10));
        list.Add(new VarItem("8", 65, 5.0, 10, 20));
        list.Add(new VarItem("9", 75, 0.4, 20, 30));
        list.Add(new VarItem("10", 35, 0.2, 0, 10));
        list.Add(new VarItem("11", 95, 1.2, 10, 20));
        list.Add(new VarItem("12", 10, 2.5, 20, 30));
        list.Add(new VarItem("13", 15, 0.6, 5, 10));
        list.Add(new VarItem("14", 85, 3.5, 5, 20));

        Console.WriteLine("Виберіть варіант 1-14 або 0 для ручного вводу:");
        Console.Write("Номер: ");
        string line = Console.ReadLine();
        int choice;
        if (!int.TryParse(line, out choice))
        {
            Console.WriteLine("Невірно, вихід.");
            return;
        }

        double a, c, x1, x2;
        if (choice >= 1 && choice <= list.Count)
        {
            var v = list[choice - 1];
            a = v.a; c = v.c; x1 = v.x1; x2 = v.x2;
            Console.WriteLine($"Вибрано {v.id}: a={a}, c={c}, x1={x1}, x2={x2}");
        }
        else if (choice == 0)
        {
            a = ReadD("a: ");
            c = ReadD("c: ");
            x1 = ReadD("x1: ");
            x2 = ReadD("x2: ");
            if (!(x2 > x1))
            {
                Console.WriteLine("x2 має бути > x1. Вихід.");
                return;
            }
        }
        else
        {
            Console.WriteLine("Невірний номер. Вихід.");
            return;
        }

        double EX = (x1 + x2) / 2.0;
        double EU = CalcEU(a, c, x1, x2);
        double CE;
        bool okCE = TryCE(a, c, EU, out CE);

        double prem = double.NaN;
        string attitude;
        if (okCE)
        {
            prem = EX - CE;
            if (prem > 1e-9) attitude = "risk-averse (premium > 0)";
            else if (Math.Abs(prem) <= 1e-9) attitude = "risk-neutral";
            else attitude = "risk-seeking (premium < 0)";
        }
        else
        {
            attitude = "CE not found";
        }

        Console.WriteLine("\n--- Результати ---");
        Console.WriteLine($"E[X] = {EX:F6}");
        Console.WriteLine($"E[U(X)] = {EU:F6}");
        if (okCE)
        {
            Console.WriteLine($"CE = {CE:F6}");
            Console.WriteLine($"Risk premium = E[X] - CE = {prem:F6}");
        }
        else
        {
            Console.WriteLine("CE не обчислено.");
        }
        Console.WriteLine($"Attitude: {attitude}");
    }

    static double ReadD(string prompt)
    {
        Console.Write(prompt);
        string s = Console.ReadLine();
        double v;
        while (!double.TryParse(s, NumberStyles.Any, CultureInfo.InvariantCulture, out v))
        {
            Console.Write("Невірно, ще: ");
            s = Console.ReadLine();
        }
        return v;
    }

    static double CalcEU(double a, double c, double x1, double x2)
    {
        double w = x2 - x1;
        if (w <= 0) throw new ArgumentException("x2>x1");
        if (Math.Abs(c + 1.0) > 1e-12)
        {
            double f = a / w / (c + 1.0);
            double t = Math.Pow(x2, c + 1.0) - Math.Pow(x1, c + 1.0);
            return f * t;
        }
        else
        {
            if (x1 <= 0) throw new ArgumentException("x1>0 for c=-1");
            double f = a / w;
            return f * (Math.Log(x2) - Math.Log(x1));
        }
    }

    static bool TryCE(double a, double c, double EU, out double CE)
    {
        CE = double.NaN;
        if (a == 0) return false;
        double ratio = EU / a;
        if (Math.Abs(c) < 1e-12) return false;
        if (ratio <= 0) return false;
        CE = Math.Pow(ratio, 1.0 / c);
        if (double.IsNaN(CE) || double.IsInfinity(CE)) return false;
        return true;
    }
}

 ```

 Результат:
 <img width="1106" height="300" alt="image" src="https://github.com/user-attachments/assets/c31f0f78-bf68-48aa-9b20-f6cf3d218fc4" />
