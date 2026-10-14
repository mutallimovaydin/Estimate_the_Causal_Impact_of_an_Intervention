# Estimate_the_Causal_Impact_of_an_Intervention

# Proposition 99-un siqaret satışlarına təsiri: Sintetik Nəzarət Metodu

## 1. Seçdiyim metod və səbəbi

**Metod:** Sintetik Nəzarət (Synthetic Control).

Niyə bu metod:

- Cəmi **bir** müalicə olunan vahid var. Klassik regressiya və ya fərqlər-fərqi (DiD) üçün tək müalicə olunan ştat və bir ədəd "orta" nəzarət qrupu kifayət qədər inandırıcı deyil.
- Uzun pre-müalicə dövrü (1970–1988, 19 il) var. Bu, 1989-dan əvvəl Kaliforniyanın trayektoriyasına uyğun çəkilər tapmağa imkan verir.
- Nəzarət qrupu subyektiv seçilmir. Çəkiləri data özü müəyyən edir.

**Necə qurdum:**

1. Datanı `year × state` formasında pivotladım (`cigsale` üzrə).
2. Pre-müalicə illərində (< 1989) Kaliforniyanın satışını donor ştatların xətti kombinasiyası ilə yaxınlaşdırdım.
3. Çəkilər üzərində məhdudiyyətlər qoydum: `w_j ≥ 0` və `Σ w_j = 1` (qabarıq kombinasiya, ekstrapolyasiya yoxdur).
4. Optimizasiya: pre-dövr orta kvadrat xətanı (MSE) minimallaşdıran SLSQP (analitik qradiyentlə). Lokal minimum riskini azaltmaq üçün 21 fərqli başlanğıc nöqtəsindən (1 bərabər çəki + 20 təsadüfi Dirichlet) başladım və ən aşağı itkini seçdim.
5. Sintetik Kaliforniya = `controls @ w`. Effekt = real − sintetik.

Optimizasiya uğurla bitdi (`Success: True`, pre-dövr MSE = **2.74**).

## 2. Fərziyyə yoxlamaları (assumption checks)

### 2.1 Pre-müalicə uyğunluğu (əsas yoxlama)

| Göstərici | Dəyər |
|---|---|
| Pre-müalicə RMSE | **1.656** |
| Pre-müalicə MAE | **1.025** |

Sintetik Kaliforniya 1989-dan əvvəl real Kaliforniyanı çox yaxından izləyir (qrafikdə iki xətt demək olar ki, üst-üstə düşür). Bu, "paralel trayektoriya" fərziyyəsinin sintetik nəzarət versiyasıdır: əgər müalicə olmasaydı, sintetik ştat Kaliforniyanın davamını yaxşı təmsil edərdi. Pre-dövr xətası post-dövr fərqindən (~19.5) təxminən 12 dəfə kiçikdir, yəni gördüyümüz boşluq uyğunlaşma xətasından açıq şəkildə böyükdür.

### 2.2 Placebo testi (Nevada)

Eyni prosedurı "saxta müalicə olunan" ştat kimi **Nevada** üzərində təkrarladım (Kaliforniya donor hovuzundan çıxarılmadı, yəni Nevada üçün hovuz 38 ştatdır).

| Göstərici | Nevada (placebo) | California |
|---|---|---|
| Pre-müalicə RMSE | 6.350 | 1.656 |
| Pre-müalicə MAE | 4.896 | 1.025 |
| Orta post-effekt | **+0.751** | **−19.514** |

Oxunuşu: real müalicə olunmayan ştatda 1989-dan sonra effekt praktiki olaraq sıfıra yaxındır (+0.75). Kaliforniyada isə effekt onun cəmi ~26 qatıdır və əks istiqamətlidir. Bu, nəticənin sistematik metod artefaktı olmadığına dair dəstəkləyici sübutdur.

### 2.3 Bu yoxlamaların məhdudiyyətləri (dürüst qeydlər)

- **Nevada üçün pre-dövr uyğunluğu zəifdir** (RMSE 6.35, Kaliforniyadakından ~3.8 dəfə pis). Ona görə bu placebo təsiri "tam təmiz" sınaq deyil, orta səviyyəli dəstək kimi qəbul edilməlidir.
- **Cəmi bir placebo** işlədilib. Rəsmi permutasiya (hamı üçün placebo) və ya post/pre RMSE nisbəti əsasında p-dəyər hesablanmayıb. Yəni nəticənin statistik əhəmiyyəti formal olaraq göstərilməyib.
- **In-time placebo** (saxta müalicə ilini daha əvvələ çəkmək) və **leave-one-out** (ən böyük çəkili donoru çıxarmaq) həssaslıq testləri aparılmayıb.

## 3. Yekun ATT qiymətləndirməsi

Post-dövr (1989–2000) illik boşluqlar (real − sintetik):

| İl | Effekt | İl | Effekt |
|---|---|---|---|
| 1989 | −8.44 | 1995 | −22.86 |
| 1990 | −9.21 | 1996 | −24.00 |
| 1991 | −12.63 | 1997 | −26.26 |
| 1992 | −13.73 | 1998 | −23.34 |
| 1993 | −17.53 | 1999 | −27.52 |
| 1994 | −22.05 | 2000 | −26.60 |

$$\widehat{ATT} = \frac{1}{12}\sum_{t=1989}^{2000}\big(Y_{CA,t} - \hat Y^{synth}_{t}\big) = \mathbf{-19.51}$$

**Yəni:** 1989–2000 ərzində Proposition 99 adambaşına siqaret satışını (data vahidi ilə, adambaşına paket) orta hesabla təxminən **19.5 vahid azaltmışdır**.

## 4. İzah (yozum)

- **İstiqamət və dinamika:** Effekt həmişə mənfidir və zamanla **güclənir**: ilk ildə −8.4, sonrakı illərdə −20 ilə −27 aralığına çatır. Bu, tədbirlərin (vergi, maarifləndirmə, məhdudiyyətlər) dərhal yox, tədricən təsir göstərməsi ilə uyğundur. Ona görə orta ATT (−19.5) son illərdəki təsirdən (≈ −26/−27) kiçik, ilk illərdən böyükdür.
- **Qrafikdə:** 1989-dan sonra real Kaliforniya xətti sintetik xəttdən aşağı ayrılır və boşluq getdikcə açılır, pre-dövrdə isə iki xətt üst-üstədir.
- **Etibarlılıq:** Yaxşı pre-dövr uyğunluğu və Nevada placebo-sunun sıfıra yaxın çıxması nəticəni dəstəkləyir.

## 5. Məhdudiyyətlər

1. **Başqa amillər:** 1989-dan sonra siqaret satışına təsir edə biləcək, datada olmayan digər dəyişikliklər (qonşu ştatlarda siyasət, ümumi tütün əleyhinə meyl və s.) effektə qarışmış ola bilər. Nəticə Proposition 99-un bütün effektinin qüsursuz sübutu deyil.
2. **Yalnız `cigsale` ilə uyğunlaşdırma:** Sintetik Kaliforniya yalnız pre-dövr siqaret satışları ilə qurulub. `lnincome`, `beer`, `age15to24`, `retprice` kimi proqnozlaşdırıcılar istifadə olunmayıb. Səbəb qismən itkin dəyərlərdir (`beer` 1209 sətirdən 663-də, `age15to24` 390-da, `lnincome` 195-də boşdur; `retprice` tam doludur). Bunları daxil etmək kontrfaktiki ssenarini daha etibarlı edə bilərdi.
3. **Statistik inferensiya yoxdur:** ATT üçün etibar intervalı və ya permutasiya p-dəyəri hesablanmayıb (bax: 3.3).
4. **Kvadrat itkinin yalnız pre-dövr üzərində minimizasiyası** çoxlu çəki kombinasiyalarının oxşar uyğunluq verməsinə (qeyri-yeganəlik) yol aça bilər. Çəkilərin sabitliyi yoxlanılmayıb.
5. **Təkrarlanma qeydi:** Notebook-da `snthetic_ca = controls.dot(weghts)` xəttində `weghts` dəyişəni heç bir görünən xanada təyin edilməyib (yalnız `best_result.x` var). Təmiz işə salma zamanı bu xəta verə bilər; düzgün forma `controls.dot(best_result.x)` olardı.
- In-time placebo və leave-one-out həssaslıq yoxlamaları.
- Əlavə proqnozlaşdırıcıları (itkin dəyərləri uyğun şəkildə işləyərək) uyğunlaşdırmaya daxil etmək.
