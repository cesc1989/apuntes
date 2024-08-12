# Lecciones RSpec - Therapists Credentialing

## Testear mailers enviados como tareas en segundo plano

Cuando intento testear el mailer `SignupMailer` desde los tests de `therapists_spec` y trato algo como:


    # 1 Ejemplo con mock visto en libro Testing Rails de Thoughtbot
    allow(SignupMailer).to receive(:signup_pdf)
    expect(SignupMailer).to have_received(:signup_pdf).with(therapist)
    
    # 2 Ejemplo de libro Every Day Testing Rails with RSpec
    expect(SignupMailer).to(receive(:signup_pdf).with(therapist: therapist))
    
    # 3 Visto en Stack Overflow https://stackoverflow.com/a/49271855/1407371
    expect {
      post "#{base_api_url}/therapists", params: { therapist: therapist }
    }.to have_enqueued_job(ActionMailer::DeliveryJob).with(
      'SignupMailer',
      'signup_pdf',
      'deliver_now',
      therapist
    )
    
    # 4 Visto en Stack Overflow https://stackoverflow.com/a/33726527/1407371
    expect {
      post "#{base_api_url}/therapists", params: { therapist: therapist }
    }.to have_been_enqueued

Ninguna resulta ya que en el controlador, el *mailer* está siendo **ejecutado en una tarea en segundo plano**.

Muchos de los ejemplos indican que se incluya los *helpers* que proporciona `ActiveJob::TestHelper` mediante:

    include ActiveJob::TestHelper

Sin embargo, el punto en contra es que esto está pensando para *minitest* y no para RSpec. En todo caso en muchos ejemplos lo usan sin problemas, aparentemente.

Relacionado:

- [ActiveJob::TestHelper#assert_enqueued_job](https://api.rubyonrails.org/v5.2.3/classes/ActiveJob/TestHelper.html#method-i-assert_enqueued_jobs)
- [Testing async emails, the rails 4.2+ way](https://www.engineyard.com/blog/testing-async-emails-rails-42)
- [How to test ActionMailer deliver_later with rspec](https://stackoverflow.com/questions/27647749/how-to-test-actionmailer-deliver-later-with-rspec)
- [How to check what is queued in ActiveJob using Rspec](https://stackoverflow.com/questions/26274954/how-to-check-what-is-queued-in-activejob-using-rspec)
- [RSpec](https://relishapp.com/rspec/rspec-rails/v/3-8/docs/matchers/have-been-enqueued-matcher) `[have_been_enqueued](https://relishapp.com/rspec/rspec-rails/v/3-8/docs/matchers/have-been-enqueued-matcher)` [y](https://relishapp.com/rspec/rspec-rails/v/3-8/docs/matchers/have-been-enqueued-matcher) `[have_enqueued_job](https://relishapp.com/rspec/rspec-rails/v/3-8/docs/matchers/have-been-enqueued-matcher)` [matchers](https://relishapp.com/rspec/rspec-rails/v/3-8/docs/matchers/have-been-enqueued-matcher)

Al final, pude *testear* con lo siguiente:


    expect {
      SignupMailer.with(therapist: therapist).signup_pdf.deliver_later
    }.to have_enqueued_job.on_queue('mailers')

Acompañado, previamente, de este *mock*:


    allow(SignupMailer).to receive(:signup_pdf)

Para que no sea tan lento el test. Visto en [Coderwall](https://coderwall.com/p/xqcq7q/how-to-test-actionmailer-activejob-with-rspec).

## Testear que correo tiene un adjunto

Visto en [Stack Overflow](https://stackoverflow.com/a/14865326/1407371):


    it 'attachs the signup PDF to the email' do
      email = SignupMailer.with(therapist: therapist).signup_pdf.deliver_now
      attachment = email.attachments.first
    
      expect(attachment).to be_a Mail::Part
      expect(attachment.content_type).to start_with('application/pdf')
      expect(attachment.filename).to include therapist.full_name
    end
## Testear Constante de una clase

Visto en [Stack Overflow](https://stackoverflow.com/questions/11337660/rails-rspec-how-to-check-for-a-model-constant).

Una de las formas sencillas es:


    describe MyClass do
      it { expect(described_class).to be_const_defined(:VERSION) }
    end

Una forma más elaborada, con un *matcher* personalizado:

En un archivo cualquiera en la carpeta `spec/support`:

    RSpec::Matchers.define :have_constant do |const|
      match do |owner|
        owner.const_defined?(const)
      end
    end

Que luego se puede usar:

    it 'defines UPLOADABLE_FILES_KEYS constant' do
      expect(described_class).to have_constant(:UPLOADABLE_FILES_KEYS)
    end


## Testear validación tipo *presence* con condicional

Cuando se usan los modificadores(`if:` o `unless:`) de una validación, los *matchers* de la gema shoulda matchers no soportan estas opciones. La forma de hacerlo lo indican en [Makandra](https://makandracards.com/makandra/46172-shoulda-matchers-how-to-test-conditional-validations).


    describe Employee do
      describe '#office' do
        context 'is a manager' do
          before { allow(subject).to receive(:manager?).and_return(true) }
          it { is_expected.to validate_presence_of(:office) }
        end
        
        context 'is not a manager' do
          before { allow(subject).to receive(:manager?).and_return(false) }
          it { is_expected.not_to validate_presence_of(:office) }
        end
      end
    end

Notar en el ejemplo del caso negativo que se usa el *matcher* con `not_to`.

## Testear carga de archivo

Antes, lo podía hacer con `fixture_file_upload` en un factory.

    wp_image { fixture_file_upload(Rails.root.to_s + '/spec/support/images/baz.jpg', 'image/jpg') }

Sin embargo, cuando volví a probar daba un error al decir que el método no existía. La recomendación es no usar dicho método ya que en realidad es un envoltorio. Mejor usar la raíz:

    trait :incomplete_file_upload do
      physical_therapy_diploma do
        Rack::Test::UploadedFile.new(Rails.root.join('spec/support/images/strawhat.png'), 'image/png')
      end
    end

Como mencionan en este [issue](https://github.com/thoughtbot/factory_bot/issues/385#issuecomment-439040731).

